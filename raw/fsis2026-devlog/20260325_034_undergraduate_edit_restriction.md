# 034. Undergraduate Students 그룹 편집 제한

**날짜**: 2026-03-25
**버전**: 0.2.42

## 개요

"Undergraduate Students" 그룹에 속한 사용자는 데이터 조회만 가능하고, 생성/편집/삭제 기능을 사용할 수 없도록 제한.

## 변경 내용

### 1. `get_user_obj()`에 `can_edit` 속성 추가

- `fsis/views.py`, `ghdb/views.py`, `kprdb/views.py`
- `user_obj.can_edit = 'Undergraduate Students' not in user_obj.groupname_list`

### 2. 뷰 레벨 가드 (URL 직접 접근 차단)

`check_can_edit()` 함수 추가 (fsis/views.py), 모든 create/edit/delete/import 뷰에 적용:

- **fsis**: litho/chrono/ref/author/sciname/journal/fig form/delete, museum_specimen form/delete/photo_delete, fossil_site add/edit/delete
- **fsis/views_import.py**: import_specimens, import_specimens_preview, import_zip, import_zip_preview
- **ghdb**: specimen_add/edit/delete
- **ghdb/views_import.py**: import_specimens, import_specimens_preview, import_zip, import_zip_preview
- **evaluator**: add/edit/delete_evaluation, add_specimen_assessment

권한 없으면 `403 Forbidden` 응답.

### 3. 템플릿 버튼 숨김 (`{% if user_obj.can_edit %}`)

총 26개 템플릿 수정:

- **fsis**: museum_specimen_list/detail, fossil_site_list/detail, author/journal/sciname/ref/chrono/litho/fig detail, author/journal list
- **ghdb**: specimen_list/detail, navbar (일괄입력 메뉴)
- **kprdb**: reference/author/journal/sciname/chrono/litho detail, reference/author/journal/sciname/litho list, chronounit_chart_sub
- **evaluator**: evaluation_detail/list, specimen_assessment_detail

### 4. CBV에 `get_context_data` 오버라이드 추가 (6개)

기존 CBV detail 뷰에 `user_obj`가 전달되지 않아 편집/삭제 버튼이 모든 사용자에게 보이던 문제 수정:

- LithoDetailView, ChronoDetailView, AuthorDetailView, ScientificNameDetailView, JournalDetailView, FigureDetailView
- ReferenceDetail 함수뷰에도 `user_obj` 컨텍스트 추가

## 수정 파일

- `config/version.py` (0.2.41 → 0.2.42)
- `fsis/views.py`, `fsis/views_import.py`
- `ghdb/views.py`, `ghdb/views_import.py`
- `kprdb/views.py`
- `evaluator/views.py`
- 템플릿 26개 (fsis 13, ghdb 3, kprdb 11, evaluator 3)

## 사용법

Django Admin에서 "Undergraduate Students" 그룹을 생성하고 해당 사용자를 그룹에 추가하면 자동 적용.
