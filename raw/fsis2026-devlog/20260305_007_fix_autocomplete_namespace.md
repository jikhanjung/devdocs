# 007: kprdb autocomplete URL 네임스페이스 수정

**작업일**: 2026-03-05

## 문제

`reference_edit2` 등 편집 페이지에서 `NoReverseMatch` 에러 발생.
`kprdb/forms.py`의 `autocomplete.ModelSelect2` 위젯에서 URL 이름에 `kprdb:` 네임스페이스 접두사 누락.

## 수정 내용

`kprdb/forms.py`에서 autocomplete URL 5개에 `kprdb:` 접두사 추가:

| 변경 전 | 변경 후 |
|---------|---------|
| `author_autocomplete` | `kprdb:author_autocomplete` |
| `journal_autocomplete` | `kprdb:journal_autocomplete` |
| `taxon_autocomplete` | `kprdb:taxon_autocomplete` |
| `lithounit_autocomplete` | `kprdb:lithounit_autocomplete` |
| `chronounit_autocomplete` | `kprdb:chronounit_autocomplete` |

## 함께 수정

- `reference_detail_body.html`: `commonname_multichoice` 표시를 `join:", "` 필터 적용하여 `['삼엽충']` → `삼엽충` 형태로 수정
