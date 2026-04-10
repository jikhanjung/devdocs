# 050. 추출 파이프라인 구축 및 Programmatic vs Claude 비교

**날짜**: 2026-03-29
**버전**: 0.2.91 → 0.2.98

## 1. 추출 스크립트 — Programmatic (`scripts/extract_from_ocr.py`)

OCR JSON 결과에서 좌표, taxa, 표본번호를 regex 기반으로 추출.

### 좌표 추출
- DMS, DM, DD, 한국어 형식 지원
- 다양한 degree/minute/second 인코딩 정규화: `°˚º◦∘`, `'′ʹ\`''`, `"″ʺ""`
- LaTeX `^{\circ}` 패턴 지원 (26건 논문 추가 발견)
- 한반도 범위 필터 (33-43°N, 124-132°E)

### 섹션 경계 감지
- "Systematic Paleontology/Palaeontology/Description" 섹션 시작 감지
- References/Acknowledgments/Discussion 등으로 섹션 종료 감지
- Taxa는 **Systematic 섹션 내에서만** 추출 (References 오탐 방지)

### Taxa 추출 (2가지 방법)
1. **이탤릭 패턴**: `<i>Genus species</i>` → binomial name
2. **Rank 패턴**: `Genus <i>Name</i>, nov.` → rank-based extraction (genus/species level만)
- 상위 rank (Family, Order 등)는 context이므로 추출하지 않음
- Nomenclatural act 분류: gen. nov., sp. nov., gen. et sp. nov., comb. nov.
- 모든 등장에서 nomenclatural act 체크 (첫 등장에서 못 잡아도 이후에 보완)

### 표본번호 추출
- 40+ 기관 prefix 지원 (KIGAM, KPE, CBNU, SNUP, USNM 등)
- 긴 prefix: 구분자 선택적 (KIGAM-9G340)
- 짧은 prefix (GL, GU 등): 구분자 필수 (오탐 방지, e.g., "Guthier" 문제 해결)

### NOT_TAXA 리스트 보강
- 저널명 (Nature, Science, Lethaia 등)
- 분류학 용어 (Generic, Material, Observations, Comparison 등)
- 지질시대명, 지명, 일반 영어 단어

### 추출 결과
| 항목 | 수치 |
|------|------|
| 좌표 | 165건 논문, 528개 위치 |
| Systematic section 감지 | 198건 |
| Taxa (섹션 제한) | 188건 논문, 6,090종 |
| 신종 | 552건 |
| 표본번호 | 139건 논문, 3,932개 |

### 한계
- Systematic section이 없는 논문 (특히 1920-1960년대 오래된 논문)은 추출 불가
- 비표준 포맷 (번호 매긴 종 기재, 섹션 헤더 없음)은 regex로 처리 어려움

## 2. 추출 스크립트 — Claude (`scripts/extract_claude.py`)

`claude -p` CLI를 사용하여 OCR 텍스트에서 LLM 기반 추출.

### 구조
- 페이지별로 OCR markdown 텍스트를 Claude에 전송
- 구조화된 JSON 응답 요청 (좌표, taxa, 표본번호)
- `{id}.pdf.claude-extract.json`으로 저장

### 옵션
- `--pages 0-4`: 특정 페이지 범위
- `--model haiku/sonnet/opus`: 모델 선택
- `--limit N`: 파일 수 제한

### 비교 결과 (3494번, pages 3-10)
| | Programmatic | Claude (haiku) |
|---|---|---|
| 좌표 | 1 | 0 |
| Taxa | 14 | 7 |
| 신종 | 3 | 1 |
| 표본 | 63 | 46 |

### 비교 결과 (2998번, pages 0-4 — 1922년 비표준 논문)
| | Programmatic | Claude (haiku) |
|---|---|---|
| Taxa | **0** (systematic section 없음) | **12** |
| 표본 | 0 | 0 |

### 평가
- **Programmatic**: 전체 페이지 커버, systematic section 있으면 정확도 높음, 빠름
- **Claude**: 비표준 포맷도 처리 가능, nomenclatural act 이해도 높음, 느리고 비용 발생
- 두 방법을 **상호보완적으로** 사용하는 것이 최적

## 3. RunPod OCR 배치 스크립트 개선 (`scripts/batch_runpod.py`)

### Health check + Wake
- 처리 시작 전 `/health` 엔드포인트로 워커 상태 확인
- idle/running 워커 없으면 wake-up 요청 후 ready까지 대기
- 타임아웃 300초 (cold start ~185초 대응)

### Payload 크기 자동 조절
- 10MiB 초과 시 `PayloadTooLarge` 예외
- 자동으로 배치를 반으로 쪼개서 재시도
- 기본 batch-size는 16 유지

## 4. PDF-DB 불일치 검사

OCR 첫 페이지 내용과 DB 메타데이터(연도, 제목) 비교.

- 검사 대상: 966건
- **3년 이상 연도 차이**: 14건 (→ devlog/20260329_049_pdf_db_mismatch.md)
- 주요 의심 사례: 2261, 2712, 2715, 3400, 3469 등

## 5. 웹 UI 변경

### pdf_status 페이지
- "산지추출" → "추출" 컬럼: 좌표/Taxa/신종/표본/SP 뱃지 표시
- 통계 카드: 추출완료, 좌표, Taxa, Syst.Paleo, 표본 필터

### pdf_detail 추출 탭
- 현재 페이지의 좌표/taxa/표본 목록
- Systematic Paleontology 섹션 위치 링크
- **Claude 추출 결과 비교**: 보라색 섹션으로 구분, 나란히 표시

### 워크스페이스 추출 패널
- 오른쪽 패널 하단에 "이 페이지 추출 데이터" 접이식 섹션
- 좌표 → 산지후보 추가 (원클릭)
- Taxa → taxon 추가 (드롭다운으로 산지 선택)
- 표본번호 → specimen 추가 (드롭다운으로 taxon 선택)
- 페이지 이동 시 자동 갱신 (전체 데이터 클라이언트 필터링)

## 수정 파일

| 파일 | 작업 |
|------|------|
| `scripts/extract_from_ocr.py` | 섹션 경계, rank 패턴, 표본번호 추출, LaTeX degree |
| `scripts/extract_claude.py` | **신규** — Claude CLI 기반 추출 |
| `scripts/batch_runpod.py` | health check, wake, PayloadTooLarge 자동 분할 |
| `kprdb/views.py` | pdf_status 추출 통계/필터, pdf_detail 추출+Claude 데이터 |
| `kprdb/views_task.py` | 워크스페이스 extract_json 전달 |
| `kprdb/templates/kprdb/pdf_status.html` | 추출 카드/컬럼/뱃지 |
| `kprdb/templates/kprdb/pdf_detail.html` | 추출 �� 표본번호, Claude 비교 |
| `kprdb/templates/kprdb/task_workspace.html` | 추출 패널, goPage 갱신 |
| `devlog/20260329_049_pdf_db_mismatch.md` | **신���** — PDF-DB 불일치 14건 |
| `config/version.py` | 0.2.91 → 0.2.98 |
