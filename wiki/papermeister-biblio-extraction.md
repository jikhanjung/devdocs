# PaperMeister LLM Bibliographic Extraction

PaperMeister **L2 Bibliographic Layer**의 핵심 기능. OCR 결과에서 제목·저자·연도·저널·DOI 같은 서지정보를 LLM으로 추출하고, 이미 검증된 Zotero 레코드 ~9,500편을 **ground truth**로 활용하여 정확도를 측정·개선한다. 2026-04-08 ~ 04-09 사이 P04/P05/015/011/012 세션에서 전체 파이프라인이 정리되었다.

## 왜 LLM인가

OCR 결과 `~/.papermeister/ocr_json/{hash}.json`의 첫 1–3 페이지에 서지정보가 있지만:

- **레이아웃이 정형적이지 않음** — 저널별, 연대별로 제목 위치 다름
- **OCR 노이즈** — 문자 깨짐, 구분자 오인식
- **아시아 언어 혼재** — 한국어/중국어/일본어 논문
- **도서/보고서/학위논문 혼재** — 논문 포맷 아닌 문서 다수

정규식/휴리스틱으로는 한계. LLM을 1차 추출기로 사용하되, **ground truth 기반 정량 평가**로 점진 개선.

## Ground Truth 전략 (P04)

현재 DB의 Paper는 세 부류:

| 구분 | 수 | 메타데이터 품질 |
|---|---:|---|
| Zotero parent item에서 import | **~9,500** | **신뢰 가능** (Zotero API, 사용자 검수) |
| Zotero standalone PDF | ~50 | 거의 비어있음 (제목만) |
| Directory source | 3 | PDF 메타데이터만, 빈약 |

**핵심 전략**: Zotero 9,500편 중 OCR 완료된 논문을 ground truth로 사용:
1. 객관적 정확도 측정 (어떤 모델/프롬프트가 강한가?)
2. 약점 파악 (아시아 언어? 책? OCR noise?)
3. 프롬프트/모델 반복 개선
4. 정확도가 충분히 높아지면 standalone PDF + directory source에 적용

이 전략의 아름다움: **Zotero 사용자의 이전 노력이 공짜 테스트셋이 된다**.

## 4-Phase 구현 (P04 → 015 → 011 → 012)

### Phase A: 평가 셋 구축

- Zotero 출처 + OCR 완료 + `Paper.title`/`authors`/`year` 모두 채워진 논문 중 **무작위 200–300편** 선정
- 다양성 보장: 영/한/중, 학술지/책/리포트, OCR 품질 high/low
- `evaluation_set` 테이블 또는 별도 JSON 파일로 관리

**메트릭**:
| 필드 | 메트릭 |
|---|---|
| Title | normalized exact match (소문자, 공백 정규화) + Levenshtein 유사도 |
| Authors | 저자 수 일치 + 순서 보존 + 이름 매칭 (성/이름 분리 고려) |
| Year | exact match |
| Journal | fuzzy match (약어/풀네임 차이 허용) |
| DOI | exact match |
| **종합** | 항목별 가중 평균 |

**Baseline**: 정규식/휴리스틱 baseline 먼저 산출 → LLM 결과와 비교 가능.

### Phase B: 추출 스키마 + 프롬프트

**Structured output 강제**:

```python
{
  "title":      str,
  "authors":    [str],              # 순서 보존, "First Last" 형식 통일
  "year":       int | None,
  "journal":    str | None,         # 책이면 publisher
  "doi":        str | None,
  "abstract":   str | None,
  "doc_type":   "article" | "book" | "chapter" | "thesis" | "report" | "unknown",
  "language":   str,                # ISO 639-1
  "confidence": "high" | "medium" | "low",
  "notes":      str | None          # LLM이 애매했던 점
}
```

**프롬프트 원칙**:
- **"추측하지 말고 본문에 명시된 것만"** 명시
- 저자 **순서 보존** 강조 (가장 자주 틀리는 항목)
- 책/논문/학위논문 등 `doc_type`을 **먼저** 판단 후 추출
- 출력은 **무조건 JSON tool call** (free-form 금지)

### Phase C: LLM 비교 실험 (011)

평가 셋 200–300편에 대해 여러 모델 비교:

| 모델 | 예상 비용 (300편) | 강점 |
|---|---|---|
| **Claude Haiku 4.5** | ~$0.25 | 빠르고 저렴 |
| **Claude Sonnet 4.6** | ~$1.5 | 어려운 케이스 정확도 |
| GPT-4o-mini | ~$0.20 | 비교군 |
| (옵션) 로컬 Ollama | $0 | 오프라인 가능성 |

**2단계 전략 검증**: Haiku 1차 → `confidence == 'low'`만 Sonnet 2차.

이 전략이 성공하면:
- 대부분의 논문 = Haiku만 사용 → 비용 최소
- 어려운 케이스 = 자동으로 Sonnet 승급 → 품질 유지
- 전체 비용 = `N × Haiku + k × Sonnet` (k ≪ N)

### Phase D: 본격 추출 + 012 교훈

실험 결과 기반으로 전체 corpus에 적용. 이 시점에서 얻은 교훈(012):

- **저자 순서 오류가 가장 흔함** — 프롬프트 reinforcement로 일부 해소
- **doc_type 오판정**이 downstream 정확도에 큰 영향 → type-specific post-processing 필요
- **언어 혼재 문서** (영어 제목 + 한국어 초록)는 language 필드를 list로 확장 고려
- **Confidence calibration**: LLM이 자기 confidence를 과대평가하는 경향 → low threshold 재조정
- **OCR 품질이 낮은 1페이지**는 2–3페이지로 fallback하는 전략이 효과적

## PaperBiblio 저장 레이어 (P04, P05)

추출 결과는 **canonical Paper를 직접 덮어쓰지 않고** `PaperBiblio` 별도 레코드로 저장:

```
Paper (canonical)
  │
  └── PaperBiblio (1:N history)
        - source: "llm_extraction"
        - model: "claude-haiku-4-5"
        - extracted_at: 2026-04-09 ...
        - confidence: "medium"
        - needs_visual_review: bool
        - fields: {title, authors, year, ...}
```

**왜 분리하나?**:
- 여러 LLM 결과를 **history로 누적** (재현성)
- **신뢰도별 정책**: `high`는 자동 반영, `medium`은 선택적, `low`는 review queue
- **needs_visual_review** 플래그 → [L5 Controlled Automation](papermeister-cli-and-gui.md)의 review queue 입력
- Canonical `Paper`는 **하나의 선택된 best source**만 반영 → 사용자가 나중에 다른 source를 승격시킬 수 있음

## Paper 반영 규칙

| 시나리오 | 동작 |
|---|---|
| Zotero에서 import된 paper | Zotero 값 유지, LLM 결과는 PaperBiblio에 기록만 (Zotero > LLM) |
| Standalone PDF + LLM high confidence | LLM 값으로 Paper 채움, Zotero write-back 후보로 등록 |
| Standalone PDF + LLM medium/low | Paper는 그대로, review queue에 추가 |
| Directory source | standalone PDF와 동일 |

이 규칙은 [sync-centric 아키텍처](papermeister-architecture.md)의 "source provenance 유지" 원칙과 일치한다 — LLM은 새로운 source가 아니라 **derived extraction**이므로 canonical Paper를 함부로 덮지 않는다.

## Vision Pass (016)

텍스트 OCR만으로 서지정보 추출이 어려운 경우(예: 제목이 이미지로만 존재, 표지 디자인이 특이한 책)를 위한 **이미지 기반 vision pass**:

- OCR text 단계에서 confidence가 낮으면 page 이미지 자체를 vision LLM에 전달
- 비용이 비싸므로 **L5 Controlled Automation의 review queue**를 통해서만 발동
- 결과는 같은 `PaperBiblio` 스키마로 저장 (`source: "llm_vision"`)

상세: devlog 016 (standalone promote + vision pass).

## 관련 페이지

- [PaperMeister 개요](papermeister-overview.md)
- [아키텍처 & 데이터 모델](papermeister-architecture.md) — `PaperBiblio`는 derived corpus layer에 속함
- [OCR 파이프라인](papermeister-ocr-pipeline.md) — 이 페이지의 입력(OCR JSON)
- [Zotero Integration](papermeister-zotero-integration.md) — Ground truth source + write-back 대상
- [CLI & GUI](papermeister-cli-and-gui.md) — P06 Desktop MVP의 L2 구현
- [제품 전략](papermeister-product-strategy.md) — LLM 비용이 MVP pricing에 미치는 영향

---
*Sources: P04 (LLM Biblio Extraction plan), P05 (Biblio Extraction Steps), 011 (LLM 모델 비교), 012 (LLM biblio 교훈), 015 (LLM biblio pipeline setup), 016 (vision pass).*
