# 046. 표본 후보 상세/편집/삭제 페이지 및 GJ PDF 임포트

**날짜**: 2026-03-28
**버전**: 0.2.86

## 1. 표본 후보 상세/편집/삭제 페이지 (v0.2.86)

`fsis.SpecimenCandidate` 모델(jpaleodb 표본 1,532건)에 대한 CRUD 페이지 추가.

### 추가 뷰
- `specimen_candidate_detail`: 전체 필드 테이블 표시, 논문 UID → reference_detail 링크
- `specimen_candidate_edit`: 모든 필드 인라인 편집 폼
- `specimen_candidate_delete`: confirm 후 삭제

### 추가 URL
- `/fsis/specimen_candidate/<int:pk>/`
- `/fsis/specimen_candidate/<int:pk>/edit/`
- `/fsis/specimen_candidate/<int:pk>/delete/`

### 템플릿
- `fsis/templates/fsis/specimen_candidate_detail.html` — 상세 (geo-detail-table)
- `fsis/templates/fsis/specimen_candidate_edit.html` — 편집 폼
- `fsis/templates/fsis/specimen_candidate_list.html` — 기재명에 상세 링크 추가

## 2. Geosciences Journal PDF 임포트 (18건)

`GeosciJournalPapers/` 디렉토리에 업로드된 PDF 18건을 저자+연도 매칭으로 DB 연결.
- GJ PDF 미보유 0건 달성
- 스펠링 차이 (`Kihm` ↔ `Khim`) 등은 유일 후보 자동 매칭으로 해결

## 3. DB 현황

| 구분 | 건수 |
|------|------|
| 전체 논문 | 1,428건 |
| PDF 보유 | 966건 (68%) |
| GJ PDF 미보유 | 0건 |

## 수정 파일

| 파일 | 작업 |
|------|------|
| `fsis/views.py` | specimen_candidate_detail, _edit, _delete 뷰 추가 |
| `fsis/urls.py` | URL 3개 추가 |
| `fsis/templates/fsis/specimen_candidate_detail.html` | **신규** |
| `fsis/templates/fsis/specimen_candidate_edit.html` | **신규** |
| `fsis/templates/fsis/specimen_candidate_list.html` | 기재명 링크 추가 |
| `config/version.py` | 0.2.85 → 0.2.86 |
