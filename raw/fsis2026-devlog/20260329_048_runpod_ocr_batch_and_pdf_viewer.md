# 048. RunPod OCR 일괄처리 및 PDF 뷰어 개선

**날짜**: 2026-03-29
**버전**: 0.2.87 → 0.2.91

## 1. RunPod OCR 일괄처리 스크립트

### scripts/batch_runpod.py (신규)
기존 `batch_runpod_seq.py`(순차 처리)를 병렬 처리로 개선.

- `--workers N` 옵션으로 동시 요청 수 설정 (RunPod max workers에 맞춤)
- `ThreadPoolExecutor`로 PDF 병렬 처리
- `/run` + poll 방식 (비동기 전송, 병렬에 적합)
- `rglob`으로 하위 디렉토리 재귀 탐색
- 기존 `.json` 결과가 있는 파일은 skip
- 증분 저장: 배치마다 결과를 즉시 저장 (중단 시 이어서 처리 가능)
- 완료 통계: complete/skip/partial/error 집계

### 실행 결과
- RunPod max workers: 8, active: 0 (서버리스)
- 모델: `datalab-to/chandra-ocr-2` (Qwen3_5 기반, vLLM 0.18.0)
- Cold start: ~185초 (모델 로딩 + torch.compile + CUDA graph capture)
- Warm 처리 속도: ~5초/페이지 (배치 16)
- 전체 966개 PDF → 153분 (2시간 33분) + 재시도 27분
- 17건 partial (10MiB body size 제한) → `--batch-size 8`로 재실행하여 해결
- **최종: 966/966 PDF OCR 완료 (100%)**

### RunPod API 제한사항
- 요청 본문 최대 10MiB
- DPI 150 기준 16페이지 배치가 10MB를 넘는 경우 있음
- 큰 페이지가 있는 PDF는 `--batch-size 8` 이하 권장

### 환경 설정
- `scripts/.env`: RUNPOD_API_KEY, RUNPOD_ENDPOINT_ID (`.gitignore` 포함)
- 의존성: PyMuPDF, Pillow, requests, python-dotenv (모두 venv에 설치 완료)

## 2. PDF 처리현황 페이지 — JSON OCR 표시 (v0.2.87)

### kprdb/views.py (`pdf_status`)
- 각 reference에 `has_json_ocr` 속성 추가 (`{pdf_path}.json` 파일 존재 확인)
- 통계에 `json_ocr` / `json_ocr_none` 카운트 추가
- `json_ocr` / `json_ocr_none` 필터 지원

### kprdb/templates/kprdb/pdf_status.html
- OCR 카드에 "JSON OCR N건" / "JSON 미처리 N건" 뱃지 추가 (클릭 필터)
- 테이블 OCR 열: JSON 파일이 있으면 파란색 "JSON" 뱃지 (기존 OCR 상태보다 우선)

## 3. PDF 상세 뷰어 — OCR 결과 캔버스 렌더링 (v0.2.87)

### kprdb/views.py (`pdf_detail`)
- `.json` OCR 결과 파일 로드 (현재 페이지 데이터)
- PDF 페이지 크기(aspect ratio용) PyMuPDF로 추출
- `json_ocr_data`, `page_size`를 JSON 직렬화하여 템플릿 전달

### kprdb/templates/kprdb/pdf_detail.html — 캔버스 렌더링
- devlog 047 가이드에 따라 구현
- PDF 실제 크기로 aspect ratio 결정 (`page_box` 미사용)
- bbox(0-1000 정규화) → 캔버스 픽셀 변환
- label별 색상 구분 (Section-Header, Text, Figure, Image, Table, Caption 등)
- 작은 bbox에서도 폰트 크기 자동 조절

## 4. 텍스트 탭 추가 (v0.2.88)

우측 패널에 레이아웃/텍스트 2개 탭 추가:

- **레이아웃 탭**: 캔버스 bbox 렌더링
- **텍스트 탭**: OCR 결과를 순서대로 읽기 좋게 표시
  - 연속 Text 청크를 하나의 문단으로 합침 (문단 구조 보존)
  - HTML 태그 보존 (`<b>`, `<sup>`, `<em>`, `<h1>` 등)
  - label별 스타일: Section-Header → 제목+구분선, Caption → italic, Table → 박스
  - Figure/Image: `<img alt="">` 설명 + `<p>` 본문 추출 표시
  - Footnote: 작은 글씨 + 좌측 보더
  - Page-Header/Footer: 작은 회색 글씨

## 5. 탭 선택 유지 (v0.2.90)

- `localStorage`로 탭 상태 저장
- 페이지 이동해도 선택한 탭(레이아웃/텍스트) 유지

## 6. OCR 라벨 전체 지원 (v0.2.91)

실제 Chandra2 OCR 라벨 확인 후 누락된 라벨 추가:
- `Figure`, `Image` (기존 `Picture` 대신)
- `Footnote`, `List-Group`, `Table-Of-Contents`

## DB 현황

| 구분 | 건수 |
|------|------|
| 전체 논문 | 1,428건 |
| PDF 보유 | 966건 (68%) |
| JSON OCR 완료 | 966건 (100%) |

## 수정 파일

| 파일 | 작업 |
|------|------|
| `scripts/batch_runpod.py` | **신규** — 병렬 OCR 일괄처리 스크립트 |
| `scripts/.env` | **신규** — RunPod 인증 정보 |
| `kprdb/views.py` | pdf_status: JSON OCR 통계/필터, pdf_detail: JSON OCR 데이터 로드 |
| `kprdb/templates/kprdb/pdf_status.html` | JSON OCR 뱃지, 필터 추가 |
| `kprdb/templates/kprdb/pdf_detail.html` | 캔버스 렌더링, 레이아웃/텍스트 탭, localStorage 탭 유지 |
| `config/version.py` | 0.2.86 → 0.2.91 |
