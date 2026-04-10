# 056. 워크스페이스 UI 개선, 연도별 추출현황 페이지, 추출 파이프라인 실행

**날짜**: 2026-04-04  
**버전**: 0.3.13 → 0.3.14

---

## 1. 워크스페이스 UI 개선

### 1.1 OCR 텍스트 탭

왼쪽 PDF 패널에 **PDF / OCR** 탭 버튼 추가:
- PDF 탭: 기존 페이지 이미지
- OCR 탭: 해당 페이지의 OCR markdown 텍스트 표시
- 페이지 이동 시 OCR 텍스트 자동 갱신
- OCR JSON이 없는 논문에는 탭 미표시
- 뷰에서 OCR JSON의 페이지별 markdown을 `ocr_json`으로 전달

### 1.2 화석산지 비고

화석산지 입력/편집/표시에 비고(remarks) 항목 추가:
- 신규 입력 폼, 편집 모드, view 모두 반영
- JS: add/edit API에 remarks 포함

### 1.3 작업 메모

워크스페이스에 메모(textarea) 추가 (입력 완료 버튼 바로 위):
- `TaskAssignment.notes` 필드 활용 (기존 모델 필드)
- debounce 자동저장 (800ms), "저장됨" 표시
- API: `api_update_notes` 추가 (`/kprdb/task_api/update_notes/<pk>/`)

### 1.4 모식표본 영어 표기

드롭다운 레이블 한글→영어 변경:
- `모식 아님` → `-`, `완모식` → `Holotype`, `부모식` → `Paratype`
- `총모식` → `Syntype`, `후모식` → `Lectotype`, `신모식` → `Neotype`

### 1.5 일반명 드롭다운

`ReferenceTaxon`에 `commonname` 필드 추가 (CharField, choices=Reference.COMMON_NAME_CHOICES):
- taxon 카드 view에 학명 옆 `(삼엽충)` 형태로 표시
- 추가/편집 폼에 드롭다운으로 선택
- `commonname_choices` 프로퍼티로 템플릿에 choices 전달

### 1.6 학명 placeholder 변경

taxon 입력란 placeholder: `종명` → `학명`

### 1.7 입력 폼 줄바꿈

taxon 추가/편집 폼에 `flex-wrap: wrap` 적용 — 공간 부족 시 자동 두 줄

### 1.8 표본 목록 접기/펼치기

taxon 카드 헤더에 ▼/▶ 토글 버튼 추가:
- 클릭 시 `taxon-card-body` (표본 목록 + 입력 폼) 접기/펼치기

---

## 2. 연도별 추출현황 페이지

`/kprdb/pdf_status_by_year/` 신규 페이지:
- 연도별 PDF 수, OCR 완료, 정규식 추출, 통합 파이프라인 현황
- 통합 파이프라인 판정: extract.json에 `figures` 키 존재 여부
- 진행률 프로그레스바, 완료(초록)/부분(노란) 행 색상
- 하단 합계 행
- 시스템관리 메뉴에 "연도별 추출현황" 링크 추가

---

## 3. 추출 파이프라인 실행

| 연도 | 건수 | 상태 |
|------|------|------|
| 2012 | 39 | 완료 |
| 2009 | 28 | 완료 |
| 2003 | 11 | 완료 |
| 1983 | 3 | 완료 |
| 1982 | 3 | 완료 |
| 1981 | 4 | 완료 |
| 1980 | 3 | 완료 |

---

## 4. 마이그레이션

- `kprdb/migrations/0011_historicalreferencetaxon_commonname_and_more.py`
  - ReferenceTaxon에 commonname 필드 추가
  - ReferenceTaxonSpecimen type_specimen choices 영어화

---

## 변경 파일 목록

- `kprdb/models.py` — commonname 필드, TYPE_SPECIMEN_CHOICES 영어화, commonname_choices 프로퍼티
- `kprdb/views.py` — pdf_status_by_year 뷰
- `kprdb/views_task.py` — commonname 처리 (add/edit), api_update_notes, OCR JSON 전달
- `kprdb/urls.py` — pdf_status_by_year, api_update_notes URL
- `kprdb/templates/kprdb/task_workspace.html` — OCR 탭, 메모, 일반명 드롭다운, 비고, 접기, flex-wrap
- `kprdb/templates/kprdb/_taxon_card.html` — 일반명 표시/편집, 접기 버튼, 영어 모식표본
- `kprdb/templates/kprdb/pdf_status_by_year.html` — 신규 템플릿
- `fsis/templates/fsisnavbar.html` — 연도별 추출현황 메뉴 링크
- `config/version.py` — 0.3.14
