# 006: 논문 목록에 PDF/Taxa/Specimens 컬럼 추가

**작업일**: 2026-03-05

## 작업 내용

kprdb 논문 목록(`reference_list.html`)에 정보 컬럼 3개 추가.

### 변경 파일

| 파일 | 변경 내용 |
|------|-----------|
| `kprdb/views.py` | `reference_list` 뷰에 `annotate(taxa_count, specimen_count)` 추가 |
| `kprdb/templates/kprdb/reference_list.html` | PDF 아이콘, Taxa 수, Specimens 수 컬럼 추가 |

### 추가된 컬럼

- **PDF**: `reference.data` 존재 시 PDF 아이콘(`bi-file-earmark-pdf`) 표시
- **Taxa**: `ReferenceTaxon` 수 (`Count('referencetaxon', distinct=True)`)
- **Specimens**: `ReferenceTaxonSpecimen` 수 (`Count('referencetaxon__specimens', distinct=True)`)
