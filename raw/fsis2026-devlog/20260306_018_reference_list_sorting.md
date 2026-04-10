# 018: 논문목록 PDF/Taxa/Specimens 컬럼 정렬 추가

## 작업 내용

`kprdb/reference_list` 페이지에서 PDF, Taxa, Specimens 컬럼 헤더 클릭 시 정렬 가능하도록 추가.

### 변경 파일

| 파일 | 변경 |
|------|------|
| `kprdb/views.py` | `Case/When`으로 `has_pdf` annotate 추가, `pdf`/`taxa`/`specimens` order_by 분기 추가 |
| `kprdb/templates/kprdb/reference_list.html` | PDF, Taxa, Specimens 헤더에 정렬 링크 + 화살표 표시 추가 |

### 정렬 기준

- **PDF**: `data` 필드 유무 (`Case(When(data='', then=0), default=1)`)
- **Taxa**: `referencetaxon` COUNT (기존 annotate 활용)
- **Specimens**: `referencetaxon__specimens` COUNT (기존 annotate 활용)
- 클릭 시 오름차순/내림차순 토글
