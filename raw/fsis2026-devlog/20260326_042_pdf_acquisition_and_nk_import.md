# 042. PDF 자동 확보 5단계 실행 및 북한 논문 임포트

**날짜**: 2026-03-26
**버전**: 0.2.74 → 0.2.79

## 1. 기존 논문 PDF 확보 (5단계)

기존 869건 중 PDF 없는 543건 대상으로 5단계 자동 확보 실행.

### 실행 결과

| 단계 | 방법 | 대상 | 확보 |
|------|------|------|------|
| 1 | DOI → Unpaywall OA | 82건 | 13건 |
| 2 | jpaleodb ejournal URL | 18건 | 10건 |
| 3 | 한국 저널 DBpia 제목 매칭 | 98건 | 36건 |
| 4 | 해외 저널 CrossRef → DOI → Unpaywall | 243건 | 1건 |
| 5 | Semantic Scholar 검색 | 339건 | 0건 |
| **합계** | | | **60건** |

기존 PDF 326건 + 60건 = 386건 (44%)

### 미확보 483건 상태 분류 (JSON 기록)
- no_lead: 189건, crossref_not_found: 102건
- paywall: ~168건 (Springer 52, Cambridge 12, T&F 7 등)
- ss_not_found: 33건, doi_found_no_oa: 30건
- 기타: ~61건

결과는 `dbpia_jkps/no_pdf_references.json`에 논문별 상태 기록.

## 2. 북한 논문 서지정보 임포트 (61건)

Oh et al. (2023) 논문의 참고문헌에서 북한 고생물학 논문 60건 + Pae et al. 1991b 1건 = **61건** 추출.

- source: `Oh et al. (2023)`
- 저자: 47명 신규, 79회 재사용
- PDF: 없음 (북한 학술지, 확보 불가)
- 한글 인용형은 remarks에 보존
- `dbpia_jkps/north_korea_paleontology_refs.json`에 서지정보 저장

### 데이터 수정
- `title_k` 비움 (한글 인용형은 제목이 아님)
- `short_title` 영문 인용형으로 재생성 (remarks에서 추출)
- `get_title` fallback 로직 추가: `language='KO'`이고 `title_k` 없으면 `title_e` 반환

## 3. UI 개선

### 대시보드 (v0.2.75)
- 논문 카드에 PDF 확보 건수 표시

### 논문 목록 필터 (v0.2.76~v0.2.77)
- 출처 드롭다운 확장: 외부 DB 제외 / 전체 / jpaleodb / 북한 논문 / 고생물학회지 / 지질학회지 / Geosciences J.
- 기본값: 전체

### Reference.get_title 개선 (v0.2.79)
- language가 KO인데 title_k가 없으면 title_e를 fallback으로 반환
- language가 EN인데 title_e가 없으면 title_k를 fallback으로 반환

## 4. 프로덕션 DB 현황

| 구분 | 건수 |
|------|------|
| 전체 논문 | 1,415건 |
| PDF 보유 | 834건 (59%) |
| 출처: 기존 | 869건 |
| 출처: dbpia | 169건 |
| 출처: jpaleodb | 175건 |
| 출처: crossref_jgsk | 88건 |
| 출처: Oh et al. (2023) | 61건 |
| 출처: crossref_gj | 53건 |

## 수정 파일

| 파일 | 작업 |
|------|------|
| `fsis/views.py` | 대시보드에 reference_pdf_count 추가 |
| `fsis/templates/fsis/dashboard.html` | PDF 건수 표시 |
| `kprdb/views.py` | 논문 목록 SOURCE_FILTERS 드롭다운 확장, 기본값 전체 |
| `kprdb/templates/kprdb/reference_list.html` | 드롭다운 동적 렌더링 |
| `kprdb/models.py` | Reference.get_title fallback 로직 |
| `config/version.py` | 0.2.74 → 0.2.79 |
| `docs/reference_import_report2.md` | **신규** — PDF 확보 및 북한 논문 보고서 |
| `dbpia_jkps/no_pdf_references.json` | 598건 논문별 PDF 확보 상태 |
| `dbpia_jkps/north_korea_paleontology_refs.json` | 북한 논문 61건 서지정보 |
| `dbpia_jkps/geosciences_journal_filtered.json` | GJ 80건 메타데이터 |
