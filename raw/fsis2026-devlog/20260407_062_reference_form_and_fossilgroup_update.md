# 062 — 논문 폼 source 필드 표시 + 화석분류군 항목 추가

**날짜**: 2026-04-07

## 변경 내용

### kprdb/forms.py
- `ReferenceForm.fields`에 `source` 필드 추가
  - 기존에 모델에는 있었으나 편집 폼에서 노출되지 않았음

### kprdb/models.py
- `Reference.FOSSILGROUP_CHOICES`에 항목 2개 추가 (마이그레이션 불필요)
  - `연체동물(mollusk)` — 이매패류/복족류/두족류 통합 상위 분류 표기용
  - `규조류(diatom)` — 신규 추가

### kprdb/templates/kprdb/reference_form2.html
- 논문 편집 폼에 `source` 필드 행 추가 (remarks 다음)

### docs/
- `pdf_missing_nk.md` — PDF 미확보 북한 관련 논문 3건 목록
- `pdf_missing_unassigned.md` — 작업 할당 없는 PDF 미확보 논문 297건 목록

## 비고
- choices 변경은 DB 스키마에 영향 없으므로 `makemigrations` 불필요
- `source` 필드는 이미 DB에 존재 (0.2.74에서 추가됨)
