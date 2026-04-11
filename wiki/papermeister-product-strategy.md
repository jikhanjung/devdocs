# PaperMeister 제품 전략 (MVP, Positioning, GTM)

`docs/` 디렉토리에는 비즈니스·제품 전략 문서가 devlog보다 더 많이 있다. 이 페이지는 그 내용을 한 곳에서 정리한다. 핵심 질문: **"기능이 다 만들어지기 전에 누가 왜 돈을 낼지"**를 먼저 검증하는 것.

## 1. Sellable MVP 정의

[sellable_mvp_definition.md](../raw/PaperMeister-docs/sellable_mvp_definition.md)의 출발점:

> 장기 비전이 크다고 해서 **모든 게 갖춰져야만 판매할 수 있는 것은 아니다.** 판매 가능한 MVP는 **지금 가장 아픈 문제 하나를 확실히 해결하는 최소 범위**여야 한다.

### 해결할 핵심 문제

**"흩어진 PDF와 reference source를 버리지 않고, 최소한 searchable corpus로 바꾸는 것"**

사용자는:
- 기존 Zotero나 폴더를 **버리지 않고**
- 스캔본과 오래된 PDF까지 포함해
- 자신의 문헌 컬렉션을 다시 찾고 활용할 수 있게 되어야 한다

### MVP의 약속

1. 기존 문헌 관리 방식을 버릴 필요가 없다
2. PDF와 스캔본을 OCR 기반으로 searchable하게 만든다
3. 한 번 corpus로 만든 자료를 이후에도 계속 다시 찾고 활용할 수 있다

이 정도면 이미 **개인 연구자와 소규모 연구실**에게 명확한 가치를 준다.

### 필수 기능 (MVP)

- source 1–2종 지원 (현실적으로 Zotero + 로컬 디렉토리)
- PDF ingest
- OCR 처리 및 캐시
- 전문 검색
- 기본 상세 뷰
- source별 browse
- 재처리 가능
- 최소 처리 안정성

### 있으면 좋은 기능

- pending / failed 처리 상태 표시
- search snippet
- source provenance 표시
- 대량 처리 안정성 보강

### 아직 없어도 되는 기능

초기 판매/파일럿 시점에서 **없어도 되는 것**:

- source 간 자동 dedup 완성도
- 완전한 canonical unified paper view
- 연구실 협업 기능 전체
- 개인/공용 annotation 공유
- entity/relation extraction
- assistant 기능
- 고급 시각화

이들은 확장 포인트이지 MVP 필수가 아니다.

### 판매 가능한 이유

> MVP의 강점은 breadth가 아니라 **즉시 체감되는 utility**다.

세 가지가 바로 체감되기 때문:
1. 이미 가진 PDF가 다시 살아난다
2. 기존 워크플로우를 부수지 않는다
3. 스캔본까지 searchable하게 된다

### 개인용 vs. 연구실용 해석

| 고객 | 포지셔닝 |
|---|---|
| 개인 연구자 | "내 PDF 아카이브를 searchable하게 해주는 도구" |
| 연구실 (파일럿) | "연구실 자료 일부를 searchable corpus로 만들어주는 파일럿 도구" |

연구실에 팔려면 최소한 설명 가능해야 할 것:
- 기존 시스템을 파괴하지 않음
- source provenance를 유지함
- 설치와 운영이 과도하게 복잡하지 않음
- OCR 비용 정책이 명확함

→ [Lab vs Individual Requirements](../raw/PaperMeister-docs/lab_vs_individual_requirements.md)에 세부 차이 정리.

## 2. PaperMeister vs RAG

흔한 질문: "그럼 RAG 같은 건가?" — 답: **아니다**. 중심이 다르다.

### 한 문장 차이

> - **RAG**는 질문에 답하기 위한 **검색층**
> - **PaperMeister**는 읽을 수 있는 **연구 corpus를 만들고 운영하는 기반층**

즉 RAG는 answer generation, PaperMeister는 corpus infrastructure.

### 일반적인 RAG 흐름

1. 문서를 넣는다
2. 문서를 chunk로 나눈다
3. embedding을 만든다
4. 질문이 들어오면 관련 chunk를 꺼낸다
5. LLM이 답을 만든다

중심: **retrieval + answer generation**. 문서는 "질문에 답하기 위한 재료"로 다뤄진다.

### PaperMeister의 중심

PaperMeister는 먼저 아래를 해결하려 한다:
- 논문이 어디에서 오는가
- PDF를 어떻게 읽을 수 있게 만들 것인가
- OCR과 layout 정보를 어떻게 보존할 것인가
- source provenance를 어떻게 유지할 것인가
- corpus를 시간이 지나도 어떻게 계속 갱신할 것인가

즉, 질문 응답 **이전**에 필요한 **source ingestion / sync / OCR / canonical corpus / provenance / 재처리 가능성**이 핵심.

### 문서를 대하는 방식 차이

**일반 RAG**: 문서를 주로 **chunk 단위 텍스트**로 다룸
- 어떤 source에서 왔는지
- 어떤 문서 구조를 가졌는지
- 어떤 파일에서 추출되었는지
- 가 상대적으로 **약하게 보존**되기 쉽다

**PaperMeister**: 문서를 **연구 자산**으로 다룸
- source, file, OCR raw result
- page, section, caption/body distinction
- provenance
- **구조와 출처를 가진 corpus asset**으로 유지

이 차이는 [아키텍처 문서](papermeister-architecture.md)의 "External = source systems, PaperMeister = canonical corpus system" 원칙으로 구체화된다.

## 3. 5-Phase Long-term Roadmap (specification)

[PaperMeister Specification](../raw/PaperMeister-docs/papermeister_specification.md)이 정의하는 장기 단계:

| Phase | 목표 |
|---|---|
| **Phase 1. Foundation Stabilization** | 현재 MVP를 운영 가능한 기반으로 다듬기 |
| **Phase 2. Structured Corpus Layer** | OCR output을 구조화 corpus representation으로 변환 |
| **Phase 3. Analysis Layer** | 코퍼스 탐색/분석 도구 (word freq, n-gram, TF-IDF, 키워드 비교, trend, 확장: taxon/time/locality 빈도, co-occurrence, 저자 네트워크) |
| **Phase 4. Entity & Assertion Layer** | 검증된 assertion 형태의 도메인 지식 추출 (taxon, formation, locality, stratigraphy, specimen, paper) |
| **Phase 5. Research Assistant** | 위의 모든 레이어 위에 쌓는 도메인 인지 agent |

**순서가 중요하다**: agent는 **마지막**이다. 좋은 corpus와 좋은 도구가 먼저 있어야 한다. [devlog 010의 pivot](papermeister-overview.md#12일-devlog-타임라인-2026-03-30--2026-04-10)에서 확인된 원칙.

**분석 우선순위 규칙** (Phase 3):
> Do not center this phase on tag clouds. Tag clouds can be added as a view, but the primary outputs should be **reusable analysis data and reports**.

## 4. Phase 1 Task Breakdown

[phase1_task_breakdown.md](../raw/PaperMeister-docs/phase1_task_breakdown.md)에 Phase 1 성공 기준과 workstream이 상세화:

### Phase 1 성공 기준

- 핵심 ingestion + OCR-derived indexing 플로우에 자동화 테스트 존재
- 일반적 failure mode가 ambiguous DB state를 남기지 않음
- 검색 결과가 highlight 등으로 검수 가능
- Batch processing + recovery paths가 검증됨
- 코드베이스가 Phase 2 schema 작업 시작하기에 충분히 안정적

### Workstream A. Test Foundation

- 테스트 러너 선택 + 설치
- `tests/` 디렉토리 구조
- 임시 DB 픽스처
- 샘플 OCR JSON 픽스처
- 헬퍼 픽스처: `Source`, `Folder`, `Paper`, `PaperFile`, `Passage`
- 로컬 테스트 실행 문서화

**우선 테스트 케이스**:
- DB 초기화가 필요한 테이블 생성
- DB migration이 기존 DB state에서 실패하지 않음
- 폴더 import가 새 PDF를 올바르게 등록
- 중복 import가 깨진 duplicate 레코드를 만들지 않음
- OCR 캐시 로딩 경로가 JSON 존재 시 작동
- 텍스트 추출이 재처리 시 idempotent
- 검색이 기본 쿼리에 대해 예상 paper/passage 반환

### Workstream B. OCR & Processing Reliability

- `pending → processed → failed` 전이 검토
- OCR + extraction의 미처리 예외 식별
- 암호화/읽을 수 없는 PDF 케이스 처리
- 손상된 OCR JSON 안전 처리
- OCR 캐시 사용 불가 시 동작 검증
- Retry 경로가 의도한 state만 reset
- Reprocess 경로의 idempotency

### Workstream C. Search UX Improvement

(docs에는 언급되지만 이 페이지에서는 Layer 4로 다룸 → [CLI & GUI](papermeister-cli-and-gui.md#l4-search-layer))

## 5. Pricing & Go-To-Market

[pricing_and_go_to_market_memo_ko.md](../raw/PaperMeister-docs/pricing_and_go_to_market_memo_ko.md)의 핵심 질문:

> PaperMeister를 판매하려고 할 때, **누가 무엇 때문에 얼마를 내고 어떤 경로로 도입할 것인가?**

지금 목표는 사업 모델 확정이 아니라 **"정말 돈을 낼 시장이 있는가"**를 큰 개발 투자 전에 확인하는 것.

### 상업적 핵심 가설

PaperMeister는 이렇게 포지셔닝하면 **안 된다**:
- 또 하나의 reference manager
- 또 하나의 PDF reader
- 또 하나의 generic AI research app

대신 이렇게 포지셔닝해야 한다:

> **기존 도구를 버리게 하지 않으면서, 흩어진 논문 라이브러리를 검색 가능하고 유지 가능한 연구 코퍼스로 바꾸는 시스템**

### 세 고객층

#### 5.1 개인 연구자

| 항목 | 내용 |
|---|---|
| 돈 낼 이유 | PDF가 흩어져 있음, 자기 아카이브를 못 찾음, OCR 기반 검색 원함, 로컬 통제 선호 |
| 장점 | 접근 쉬움, 피드백 빠름, 초기 검증에 좋음 |
| 약점 | 예산 작음, 가격 민감, 무료 도구와 비교 잦음 |

#### 5.2 연구실 / 연구 그룹

| 항목 | 내용 |
|---|---|
| 돈 낼 이유 | 학생/포닥/Zotero/폴더 등에 논문 분산, 연구실 문헌 자산 조각화, 새 구성원 인수인계 어려움 |
| 장점 | 지불 의사 강함, pain point 분명, multi-source 포지셔닝에 잘 맞음 |
| 약점 | 의사결정 느림, 설치/신뢰 중요, 지원 기대치 높음 |

#### 5.3 기관 & 특수 팀

예: 대학 연구센터, 박물관, 기업 R&D 팀, 도메인 아카이브 조직

| 항목 | 내용 |
|---|---|
| 돈 낼 이유 | 내부 문헌 인프라 필요, 장기 corpus 자산 중요, 설치 지원/마이그레이션/커스터마이징 |
| 장점 | 계약 단가 높음, 도메인 적합 시 강력한 use case |
| 약점 | 판매 주기 길음, 구매 절차 복잡, 지원·배포 부담 큼 |

### Customer Discovery

[customer_discovery_plan.md](../raw/PaperMeister-docs/customer_discovery_plan.md) + Korean version: 인터뷰 target 선정, 질문지 설계, pilot 제안서 작성. 영/한 병행 자료 구비.

### 파일럿 제안 자료

- **`pilot_one_pager_ko.md`** — 1페이지 파일럿 제안
- **`interview_outreach_email_ko.md`** — 인터뷰 요청 이메일 초안
- **`landing_page_copy.md` + `_ko.md`** — 초기 랜딩 페이지 카피
- **`leaflet_copy.md`** — A4 2–3장 분량 소개 카피

## 6. AI 시대 학술 인프라 Positioning

[ai_era_academic_infrastructure_positioning.md](../raw/PaperMeister-docs/ai_era_academic_infrastructure_positioning.md)의 포지셔닝 메모:

- PaperMeister는 단일 AI 앱이 아니라 **"AI 시대의 학술 인프라"**
- LLM/에이전트는 PaperMeister 위에 **쌓이는 레이어**이지, PaperMeister를 대체하지 않는다
- corpus, provenance, 재현성은 agent 시대에 **더 중요해진다** (덜 중요해지는 게 아님)
- 이 관점이 [Noematica 회사 브랜드](noematica-brand.md)의 "scholarly infrastructure company" 아이덴티티로 발전

## 7. OCR 비용과 pricing의 연결

[OCR Cost Optimization](papermeister-ocr-pipeline.md#비용-최적화-핵심-원칙)의 전략은 **가격 모델과 직접 연결**:

- 개인: pre-filter + priority OCR → OCR 비용을 사용량으로 부담 (on-demand)
- 연구실: reserved GPU + 선택적 OCR → 월정액 또는 프로젝트 기반 과금
- 기관: 내부 설치 + 커스텀 OCR backend → 연간 라이선스 + 지원 계약

이 계층화는 [GTM의 세 고객층](#5-pricing--go-to-market)과 자연스럽게 맞물린다.

## 관련 페이지

- [PaperMeister 개요](papermeister-overview.md)
- [아키텍처 & 데이터 모델](papermeister-architecture.md)
- [OCR 파이프라인](papermeister-ocr-pipeline.md) — 비용 전략의 구현
- [LLM Biblio Extraction](papermeister-biblio-extraction.md) — Ground truth 전략
- [Zotero Integration](papermeister-zotero-integration.md) — 첫 번째 target 고객의 주 source
- [CLI & GUI](papermeister-cli-and-gui.md) — P06/P07 feature scope 구현
- [Noematica 브랜드](noematica-brand.md) — "scholarly infrastructure" 정체성

---
*Sources: docs/papermeister_specification.md, docs/sellable_mvp_definition.md, docs/papermeister_vs_rag.md, docs/phase1_task_breakdown.md, docs/lab_vs_individual_requirements.md, docs/ai_era_academic_infrastructure_positioning.md, docs/pricing_and_go_to_market_memo.md + _ko, docs/customer_discovery_plan.md + _ko, docs/landing_page_copy.md + _ko, docs/pilot_one_pager_ko.md, docs/interview_outreach_email_ko.md, docs/leaflet_copy.md.*
