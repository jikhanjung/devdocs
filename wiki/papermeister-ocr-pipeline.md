# PaperMeister OCR 파이프라인 & 비용 최적화

PaperMeister의 **L2 Corpus Layer** 핵심 작업. PDF를 받아 layout 정보를 보존한 구조화 텍스트로 변환하고, 그 결과를 손실 없이 캐싱하여 이후 모든 파생 레이어(passage, bibliographic, entity, assertion)가 재현 가능한 형태로 사용할 수 있게 한다. RunPod Chandra2를 기본 엔진으로 쓰면서 비용을 통제하는 전략이 이 페이지의 두 번째 축이다.

## 파이프라인 개요

```
PDF 파일
   │
   ▼
[1] PyMuPDF 텍스트 레이어 추출         ← ingestion.py / text_extract.py
   │   - 이미 텍스트 레이어가 있는가?
   │   - 품질이 사용할 만한가?
   │   - 제목/초록 추출 가능한가?
   │
   ├── 충분함 → OCR 건너뛰기, 직접 passage 생성
   │
   └── 부족함 → [2]
          ▼
[2] RunPod Chandra2 OCR              ← GPU 기반
       - section / caption / body / layout 구조 인식
       - page 단위 결과
   │
   ▼
[3] Raw OCR JSON 캐시                 ← ~/.papermeister/ocr_json/{hash}.json
       - 불변 source-of-truth
       - 재처리/재분석에 재사용
   │
   ▼
[4] Passage 분할 + FTS5 인덱스        ← search.py
       - BM25 랭킹
       - snippet() 하이라이트
       - metadata: title(×10), authors(×5), text(×1)
```

## 왜 Chandra2인가

[OCR 비용 최적화 전략 문서](#)(ocr_cost_optimization_plan.md)의 핵심 관찰:

> Chandra2의 가치는 단순한 텍스트 인식률이 아니다. 특히 PaperMeister에서는:
> - 높은 OCR 정확도
> - **section / caption / body 구분**
> - **figure / table 주변 정보 인식**
> - **layout 구조 보존**

즉 Chandra2는 단순 OCR 엔진이 아니라 **구조화 가능한 코퍼스를 만드는 엔진**이다. 이것이 [Phase 2 Structured Corpus Layer](papermeister-overview.md#장기-5-phase-로드맵)의 전제 조건이기 때문에, 단순히 "가장 싼 OCR로 교체"는 제품 방향과 맞지 않는다.

## 병렬 OCR 처리 (004, P03)

MVP 구현 직후(03-30) 순차 OCR의 한계가 드러나 병렬 처리 도입:

- **P03**: 병렬 OCR 계획 — 처리 큐, concurrency 제어, 상태 전이 설계
- **004**: 구현 — 여러 PDF를 동시에 Chandra2에 보내고, 결과 수집 순서는 파일 독립적으로 관리
- 각 파일의 상태: `pending` → `processing` → `processed` | `failed`

## Ollama GLM-OCR 대안 실험 (005)

로컬/오프라인 가능성을 탐색하기 위해 Ollama 경유 GLM-OCR 모델 테스트. 결과는 **Chandra2 대체가 아닌 보완 후보**로 남음 — 품질과 layout 인식 측면에서 Chandra2만큼의 structured output을 제공하지는 못했지만, cost-free 옵션이 필요할 때의 fallback으로 기록.

→ 이 실험은 [P07 Desktop 구현 계획](papermeister-cli-and-gui.md)의 "Layer 6 Hardening — backend switching" 항목의 근거가 된다.

## Batch Retry / Resume / Concurrency (018, 2026-04-09)

대량 처리 시 안정성 보강:

### 문제
- 수백 편 처리 중 네트워크/GPU 실패가 일부 파일을 `failed` 상태로 남기고, 전체 배치를 다시 돌리면 이미 `processed`된 것까지 재처리
- 동시 처리 파일 수가 너무 많으면 RunPod 한도 초과
- Crash 복구 시 어디부터 재개할지 불명확

### 해결
- **재시도 정책**: `failed` 상태에 실패 이유 저장 → 자동 재시도 vs. 수동 검토 분기
- **Resume**: 이미 `processed`인 파일은 skip, `pending`/`failed`만 다음 배치에 포함
- **Concurrency 제어**: 동시 처리 파일 수를 설정 가능한 상한으로 제한
- **실패 분류 (failure taxonomy)**:
  - 일시적 (네트워크, rate limit) → 자동 재시도
  - 영구적 (암호화 PDF, 손상 파일) → 사용자 검토 큐로 이동
  - 알 수 없음 → 로그 + 수동 확인

→ 이 작업의 결과가 [P07 Layer 6 Hardening](papermeister-cli-and-gui.md) 레이어로 정식화된다.

## 비용 최적화 핵심 원칙

**"더 싼 OCR을 찾는 것보다, 고품질 OCR을 정말 필요한 PDF와 페이지에만 쓰는 것"**

### 1. OCR 필요 여부 판별 (pre-filter)

모든 PDF를 똑같이 OCR하지 않는다. PyMuPDF로 먼저 텍스트 레이어를 검사:

- 추출 텍스트 길이
- 페이지당 텍스트 분포
- 문자 깨짐 정도
- 제목/초록 추출 가능 여부

이 기준을 통과하지 **못하는** PDF만 Chandra2로 보낸다.

### 2. 선택적 OCR (priority-based)

컬렉션 전체를 한 번에 완주하지 않고 우선순위를 둔다:

- 자주 참조하는 컬렉션 우선
- 현재 프로젝트 관련 논문 우선
- 스캔본 / OCR-poor PDF 우선
- 최근 추가된 논문 우선

비용을 **전체 coverage**가 아닌 **perceived usefulness**에 먼저 배치.

### 3. 페이지 단위 선택 처리

가능하면 PDF 전체가 아니라 **필요한 페이지만** OCR:
- 텍스트가 거의 없는 페이지만
- 서지정보가 있는 첫 1–3 페이지 먼저
- 이미지/figure 중심 페이지는 나중에

## Reserved GPU 비용 분석 (017, 2026-04-09)

장기 대량 처리 시 on-demand vs. reserved GPU 비용 비교:

- **On-demand**: 처리 시 매번 과금, idle 비용 없음, 시작 지연 (cold start)
- **Reserved**: 시간 단위 고정 비용, idle 시에도 과금, 즉시 사용 가능

손익 분기: **지속적으로 대량 처리할 때만 reserved가 유리**. 개인/초기 파일럿 단계에서는 on-demand + pre-filter + priority 전략이 더 저렴.

→ 이 분석은 [제품 전략의 GTM](papermeister-product-strategy.md)에서 "개인 vs. 연구실 vs. 기관"별 OCR 비용 정책 설계의 근거가 된다.

## OCR JSON 캐시 구조

`~/.papermeister/ocr_json/{hash}.json` — PDF 해시를 key로 사용:

- **불변**: 한 번 생성되면 재OCR 전까지 변경 없음
- **재현 가능**: 모든 derived layer(passage, bibliographic extraction, entity extraction)가 이 JSON에서 재생성 가능
- **source-of-truth**: corpus의 근본 자산. 백업 대상 1순위.

```json
{
  "hash": "sha256:...",
  "engine": "chandra2",
  "pages": [
    {
      "page_num": 1,
      "blocks": [
        {"type": "section_header", "text": "...", "bbox": [...]},
        {"type": "body", "text": "...", "bbox": [...]},
        {"type": "caption", "text": "...", "bbox": [...]},
        ...
      ]
    },
    ...
  ],
  "metadata": {
    "processing_time_sec": 12.3,
    "chandra2_version": "...",
    ...
  }
}
```

이 구조는 [LLM Bibliographic Extraction](papermeister-biblio-extraction.md)의 입력이 되고, 향후 [Phase 2 Structured Corpus Layer](papermeister-overview.md)의 직접 입력이기도 하다.

## Demo Corpus & 저작권 정책

라이선스 불명확한 PDF를 corpus에 담을 수 없으므로 **데모용 curated corpus** 전략이 별도로 필요:

- **Public domain** 학술 문헌만 demo에 포함 (예: 19–20세기 초 paleontology monograph)
- OCR 결과를 데모 파일로 함께 배포 가능
- 사용자가 자기 PDF를 올릴 때는 **로컬 처리만** (네트워크 전송은 Chandra2 GPU로만)

→ 상세: docs/demo_corpus_strategy.md, docs/copyright_demo_policy.md

## 관련 페이지

- [PaperMeister 개요](papermeister-overview.md) — 전체 파이프라인 맥락
- [아키텍처 & 데이터 모델](papermeister-architecture.md) — OCR 결과가 canonical Paper에 귀속되는 이유
- [LLM Bibliographic Extraction](papermeister-biblio-extraction.md) — OCR JSON의 주 소비자
- [CLI & GUI](papermeister-cli-and-gui.md) — P07 Layer 6 Hardening의 배경
- [제품 전략](papermeister-product-strategy.md) — OCR 비용이 가격 모델에 미치는 영향

---
*Sources: 002 (OCR pipeline), 004 + P03 (parallel OCR), 005 (Ollama GLM-OCR), 017 (reserved GPU 비용), 018 (batch retry/resume/concurrency), docs/ocr_cost_optimization_plan.md, docs/demo_corpus_strategy.md, docs/copyright_demo_policy.md.*
