# KPRDB (Korean Paleontology Reference Database)

한국 고생물학 논문 정보를 관리하는 Django 앱. FSIS2026의 핵심 데이터 관리 모듈이다.

## 주요 모델

| 모델 | 역할 |
|---|---|
| **Reference** | 논문 정보 (제목, 저자, 저널, 연도, DOI, PDF, UID 등) |
| **Author** | 저자 정보 |
| **Journal** | 저널/학술지 (soft-delete 지원, is_deleted 플래그) |
| **ReferenceTaxon** | 논문에서 추출된 분류군 |
| **ReferenceTaxonSpecimen** | 분류군별 표본 정보 |
| **FossilSiteReference** | 화석산지-논문 연결 (junction 모델) |
| **SpecimenCandidate** | jpaleodb 임포트 표본 후보 (source='jpaleodb') |

## Reference 모델 주요 필드

- **UID**: `kprdb:bib:{source}:sha256:{hash}` 패턴, 충돌 감지 및 경고 시스템
- **source**: 논문 출처 (jpaleodb, crossref_jgsk, crossref_gj, dbpia, YJOh 등)
- **type**: 문헌 유형 — JN(저널), BK(도서), BS(도서 섹션), RP(보고서), TH(학위논문)
- **extracted_data**: JSONField, PDF에서 추출한 좌표/분류군/표본 데이터
- **title_k / title_e**: 한국어/영어 제목 (언어별 fallback 로직)
- **도서 전용 필드**: book_title, publisher, place, series, isbn, url 등 9개

## 문헌 유형별 UI

문헌 유형에 따라 폼 필드가 동적 표시:
- JN/RP/TH → 저널 관련 필드
- BK/BS → 도서 관련 필드
- 아이콘: book (BK/BS), file-text (JN/RP), mortarboard (TH)

## 논문 목록 기능

- **정렬**: PDF 유무, Taxa 수, Specimens 수 (Case/When ORM)
- **필터**: source 드롭다운, PDF 유무, 화석분류군
- **고급 검색**: 연도 범위, 저자/제목/저널 텍스트, 문헌유형, PDF 상태, 화석분류 체크박스
- **페이지네이션**: 인라인 paginator, author_detail/journal_detail에도 적용 (20건/페이지)
- **컬럼**: author+year 병합, 언어별 저널명 표시, 압축 헤더 (Taxa→Tx, Specimens→Sp)

## 학명 이탤릭 렌더링 (068, 2026-04-11)

`scientific_name` 필드에 저장된 `<i>...</i>` 태그가 실제 이탤릭으로 표시되도록 템플릿에 `|safe` 필터 적용. 적용 파일 5개:
- `evaluator/type_specimen_list.html`
- `evaluator/specimen_assessment_detail.html`
- `evaluator/specimen_assessment_form.html`
- `evaluator/site_specimen_list.html`
- `fsis/fossil_site_detail.html`

## 저널 목록 논문 수 표시 (068)

`journal_list` 뷰에 `annotate(ref_count=Count('reference'))` 추가. `journal_list.html` 템플릿에 "논문" 컬럼(60px, 중앙 정렬) 신규. → 저널별 출판 편수를 한눈에 확인 가능. 저널 데이터 정리(dedup/merge) 작업과 함께 수행되어 정리 효과를 가시화. 상세 병합 목록은 [데이터 수집 > 저널 데이터 정리](data-import.md#저널-데이터-정리-068-2026-04-11) 참조.

## 저자 관리

- 드래그앤드롭 저자 순서 변경 (HTML5 drag-and-drop, grip 아이콘)
- 커스텀 autocomplete 드롭다운 + 신규 저자/저널 생성 모달
- 북한 논문 저자 매핑 (38명 업데이트)

## FOSSILGROUP_CHOICES

화석분류군 선택지: mollusk(연체동물, 이매패/복족/두족 통합), diatom(규조류) 등

## 관련 페이지

- [PDF 처리 파이프라인](pdf-pipeline.md) — 논문 PDF에서 데이터 추출
- [데이터 수집](data-import.md) — 외부 소스에서 논문 임포트, 저널 정리 상세
- [북한 논문 데이터](north-korea-data.md) — NK 저자 한글 매핑 출처
- [작업 관리](task-management.md) — 논문 처리 작업 할당
- [KOFHIN 매뉴얼](kofhin-manuals.md) — 논문 등록 / 검토 워크플로우

---
*Sources: 001, 006, 007, 029, 031, 032, 037-044, 062, 064, 066, 067, **068**, docs/admin_manual.md, docs/user_manual_data_entry.md.*
