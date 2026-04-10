# 057. PDF 확보 작업 단계, 작업할당 UI 개선, 다국어/원어 표기 정리

**날짜**: 2026-04-04 ~ 2026-04-05  
**버전**: 0.3.14 → 0.3.16

---

## 1. PDF 확보 작업 단계 추가

### 1.1 Phase 구조 변경

기존 2단계에서 3단계로 확장:
- Phase 1: **PDF 확보** (신규)
- Phase 2: **자료 입력** (기존 phase 1)
- Phase 3: **입력자료 검토** (기존 phase 2)

상수 정의: `PHASE_PDF=1`, `PHASE_ENTRY=2`, `PHASE_REVIEW=3`  
코드 전체에서 하드코딩 숫자 대신 상수 사용.

### 1.2 데이터 마이그레이션

`0012_shift_phase_numbers.py`: 기존 phase 값을 2→3, 1→2 순으로 이동.  
프로덕션 DB에 직접 적용 (Docker 배포 전 호스트에서 실행).

### 1.3 PDF 확보 상태

새 상태 추가:
- `pending_pdf` (PDF확보 대기)
- `finding_pdf` (PDF확보 중)
- `pdf_complete` (PDF확보 완료)
- `pdf_unavailable` (PDF확보 불가) — 빨간 badge

### 1.4 검토자 메모

검토자(phase 3) 워크스페이스에서 입력자(phase 2)의 메모를 readonly로 표시.

---

## 2. PDF 업로드 페이지

`/kprdb/my_pdf_uploads/` 별도 뷰/템플릿:
- 논문 정보 테이블 (저자, 연도, 제목, 저널, 페이지)
- 행 높이 160px, 드래그앤드롭 드롭존
- 업로드 성공 시 PDF 첫 페이지 썸네일 표시, 상태 자동 변경
- Google Scholar 검색 버튼 (저자+제목+연도)
- 상태 드롭다운 (확보중/확보불가/확보완료) + 메모 입력 (debounce 자동저장)

내 작업 페이지에서 PDF 확보 섹션 → "PDF 업로드" 버튼으로 연결.

---

## 3. 작업할당 페이지 개선

### 3.1 AJAX 논문 목록

논문 목록을 AJAX로 로드하도록 변경 (`_task_add_reflist.html` 파편 템플릿):
- 필터 변경, 페이지네이션 시 작업자/단계/비고 유지
- 이벤트 위임으로 동적 콘텐츠 지원

### 3.2 필터 추가

- "PDF 없는 미할당만" 필터 (`show_mode=no_pdf`)
- 기본 정렬: 연도 역순 (`-year`)
- 작업 단계 변경 시 자동 전환: PDF 확보→PDF없는미할당, 자료입력→미할당, 검토→입력완료

### 3.3 논문 정보 툴팁

제목 hover 시 팝업: 저자, 연도, 영문제목, 원어제목(다를 때만), 저널(권/호/페이지), DOI, 언어, 비고.  
`journal` FK가 None인 경우 500 에러 수정 — 뷰에서 `display_journal` 사전 계산.

### 3.4 작업 목록 테이블

PDF대기/PDF확보중/PDF완료/PDF불가 컬럼 추가.

---

## 4. 다국어/원어 표기 정리

### 4.1 논문 (Reference)

- `title_k` verbose_name: "한글제목" → "원어제목"
- `language` 선택지 추가: Japanese, Chinese, Russian, German, French

### 4.2 저자 (Author)

- `firstname_k`/`lastname_k` verbose_name: "이름"/"성" → "원어 이름"/"원어 성"
- 템플릿: "한글이름" → "원어이름", "성(한글)/이름(한글)" → "성(원어)/이름(원어)"
- `generate_short_title()`: 비영어 논문에서 `abbreviation_k` 없을 때 `lastname_e`로 fallback (None 표시 문제 해결)
- 비영어 논문 67건 short_title 재생성

### 4.3 저널 (Journal)

- `title_k`→"원어제목", `title_e`→"영문제목", `publisher`→"출판사", `since`→"창간연도"
- journal_detail: 영문 레이블 → 한국어

---

## 5. OCR cron 개선

- incremental 모드의 `-newer` 타임스탬프 방식 제거
- 항상 JSON 없는 PDF를 찾도록 단순화 (타이밍 문제 해결)
- `entrypoint.sh`에 `umask 002` 추가 (Docker에서 생성한 디렉토리에 group 쓰기 권한)
- 기존 디렉토리 전체 `chmod g+w` 적용

---

## 6. 학술대회/초록 논문 작업 할당

비고에 학술대회/학술발표/초록/abstract 포함 논문 81건 → admin에 PDF 확보 작업 일괄 할당.

---

## 7. 추출 파이프라인 실행

| 연도 | 건수 |
|------|------|
| 1977 | 8 |
| 1978 | 1 |
| 1979 | 4 |

---

## 마이그레이션

- `0011`: ReferenceTaxon.commonname 추가
- `0012`: phase 번호 이동 (1→2, 2→3)
- `0013`: phase/status choices 변경
- `0014`: pdf_unavailable 상태 추가
- `0015`: Reference.language 선택지, title_k verbose_name
- `0016`: Author firstname_k/lastname_k verbose_name
- `0017`: Reference.title_k "원어제목"
- `0018`: Journal 필드 verbose_name
- `0019`: Journal.since "창간연도" (인코딩 수정)

---

## 변경 파일 목록

- `kprdb/models.py` — phase/status 구조, commonname, 다국어 verbose_name
- `kprdb/views_task.py` — PDF 확보 뷰, AJAX 논문목록, display_journal, 메모 API
- `kprdb/views.py` — pdf_status_by_year
- `kprdb/urls.py` — my_pdf_uploads, api_update_notes, pdf_status_by_year
- `kprdb/templates/kprdb/task_workspace.html` — OCR 텍스트 탭, 메모, 접기, 일반명 등
- `kprdb/templates/kprdb/_taxon_card.html` — 일반명, 접기, 영어 모식표본
- `kprdb/templates/kprdb/task_assignment_add.html` — AJAX, 툴팁, 필터
- `kprdb/templates/kprdb/_task_add_reflist.html` — AJAX 파편 (신규)
- `kprdb/templates/kprdb/task_assignment_list.html` — PDF 상태 컬럼
- `kprdb/templates/kprdb/my_tasks.html` — PDF 확보 섹션
- `kprdb/templates/kprdb/my_pdf_uploads.html` — PDF 업로드 페이지 (신규)
- `kprdb/templates/kprdb/pdf_status_by_year.html` — 연도별 현황 (신규)
- `kprdb/templates/kprdb/author_*.html` — 원어이름 표기
- `kprdb/templates/kprdb/journal_detail.html` — 한국어 레이블
- `fsis/templates/fsisnavbar.html` — 연도별 추출현황 메뉴
- `scripts/extract_from_ocr.py` — 단일 파일 모드, parse_raw_to_lat_lng
- `scripts/ocr_cron.sh` — JSON 없는 PDF 직접 탐색
- `deploy/entrypoint.sh` — umask 002
- `config/version.py` — 0.3.16
