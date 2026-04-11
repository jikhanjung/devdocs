# 데이터 수집 (외부 소스 임포트)

다양한 외부 소스에서 고생물학 논문과 표본 데이터를 수집하는 과정.

## 소스별 임포트 현황

| 소스 | 유형 | 논문 수 | PDF 수 | 비고 |
|---|---|---|---|---|
| **jpaleodb** | 웹 스크래핑 | 177 | 30 | J-STAGE PDF, 표본 1,532건 |
| **DBpia** | API 리버스엔지니어링 | 412 (55개 호) | 368 | 한국고생물학회지 1985-2014 |
| **CrossRef (JGSK)** | API 검색 | 140 → 88 신규 | 126 (DOI→DBpia) | 대한지질학회지 ISSN 0435-4036 |
| **CrossRef (GJ)** | API 검색 | 80 → 53 신규 | 1 (OA) | Geosciences Journal, Springer 페이월 |
| **YJOh** | 수동 입력 | 61 | 96 | Oh et al.(2023) 북한 고생물학 논문 |
| **KPRDB** | DB 임포트 | — | — | 화석산지 50건 |

## 5단계 PDF 확보 파이프라인

1. **Unpaywall OA**: 오픈 액세스 PDF 확인
2. **jpaleodb**: J-STAGE + handle.net PDF 다운로드 (retry 로직)
3. **DBpia**: API 기반 PDF 다운로드
4. **CrossRef**: DOI → DBpia nodeId 매핑
5. **Semantic Scholar**: 추가 OA PDF 탐색

**결과**: 834/1,415 (59%) → 948/1,428 (66%) PDF 확보

## jpaleodb 임포트 상세

- jpaleodb.jp 웹사이트에서 Korea/Chosen 관련 논문 177건 + 표본 1,532건 스크래핑
- import_jpaleodb 관리 커맨드: 저자 파싱, 저널 매칭
- SpecimenCandidate 모델 생성 (source='jpaleodb', detail_url 링크)
- CRUD 페이지: 상세/편집/삭제 뷰

## DBpia 임포트 상세

- DBpia API 리버스엔지니어링: voisListAjax, journal/voisNodeList
- PDF 다운로드 엔드포인트 발견
- 55개 호, 412건 논문 중 368건 PDF 다운로드 (44건 비논문 섹션 필터링)
- 메타데이터 JSON + PDF 파일 (nodeId 기준 정리)

## CrossRef 임포트 상세

- ISSN 기반 검색
- 고생물학 키워드 필터링 (312 → 140건 for JGSK)
- DOI → DBpia nodeId 매핑으로 PDF 확보
- Unpaywall OA 감지 (GJ: Springer 페이월로 1건만 성공)

## 북한 논문 임포트

- Oh et al. (2023) 참고문헌 61건 (source='YJOh')
- nkpaleo/ 디렉토리에서 96건 PDF 임포트
  - 한국어 파일명 매칭: 55건 기존 레코드, 32건 신규 생성
  - 1:N 매칭 해결
  - 23명 신규 저자 생성
- 4건 도서 PDF: 조선의화석1/2, 조선고생물화석, Sinuiju Biota
- 저자 매핑: 38명 업데이트, 13명 미매칭

## KPRDB 화석산지 임포트

- 50건 화석산지 임포트
- site_code 자동 생성
- FossilSiteReference 연결 모델
- Nominatim 역지오코딩

## UID 체계

- 패턴: `kprdb:bib:{source}:sha256:{hash}`
- 초기 접두사: `scoda:bib:` → `kprdb:bib:`로 변경
- 충돌 감지 + 경고 시스템 (프론트엔드 노란색 배지)
- generate_uids 관리 커맨드

## PDF-DB 불일치 관리

- no_pdf_references.json: 확보 실패 문서화
- 14건 의심 불일치 감지 (OCR 메타데이터 vs DB 비교)
- 중복 논문 5그룹 식별
- 잘못 연결된 PDF 교정

## 수집 방법 메모 (docs 보고서 참고)

`docs/reference_import_report.md` / `reference_import_report2.md`는 **2026-03-24 ~ 03-26 시점의 일회성 잡 기록**이다. 집계 수치(당시 1,354건 / 756건 등)는 **스냅샷일 뿐 현재 상태가 아니다** — 최신 DB 규모는 [overview.md의 데이터 규모](overview.md#데이터-규모-2026-04-11-기준) 참조. 이 섹션은 **방법론**만 정리한다.

### Unpaywall 기반 OA 탐색

- DOI가 있는 논문에 대해 Unpaywall API 호출
- J-STAGE OA의 경우 `/_article/` URL을 `/_pdf/` 로 치환하여 직접 다운로드 (Unpaywall이 paywall로 오판하는 경우에도 실제로는 OA인 경우 있음)
- 실패 분류: Springer paywall, jgsk.or.kr paywall, Elsevier/BioOne, OA이지만 PDF URL 없음

### ejournal URL 직접 접근

- jpaleodb 논문의 `ejournal_url` 필드를 활용
- 성공 도메인: `umdb.um.u-tokyo.ac.jp` (도쿄대 박물관 Kobayashi 시리즈), `dinosaur.pref.fukui.jp` (후쿠이 공룡박물관)
- 실패 도메인: `hdl.handle.net` (인증/404)

### DBpia API 리버스엔지니어링

- 내부 엔드포인트: `POST /voisListAjax`, `POST /journal/voisNodeList`, `GET /pdf/pdfView.do?nodeId=`
- 호(volume) 목록 → 논문 목록 → PDF 순차 수집

### CrossRef ISSN 검색 + DOI→DBpia 매핑

- ISSN 기반 학술지 검색
- 고생물학 키워드로 필터링
- DOI를 DBpia nodeId로 매핑하여 DBpia API로 PDF 다운로드

### PDF 미확보 목록 (역사 자료)

`docs/pdf_missing_unassigned.md` (297건, 연도별) · `docs/pdf_missing_nk.md` (3건, NK). 모두 **2026-04-07 기준 스냅샷**이며 이후 일부 확보되었을 수 있음.

## 저널 데이터 정리 (068, 2026-04-11)

프로덕션 DB에서 중복/오타 저널 병합:

| 병합 전 | 병합 후 | 건수 |
|---|---|---|
| IGCP-350 저널 5개 (625/629/630/631) | **628** (통합) | 6건 |
| "Geosciences Journa" (오타, 459) | **451** "Geosciences Journal" | 9건 |
| "ICHNOS-AN INTERNATIONAL..." (563) | **435** "Ichnos" | 4건 |
| "Journal of paleontology" 소문자(430) | **428** "Journal of Paleontology" | 2건 |
| 지구과학회지 3중(427/431/513) | **470** (영문+원어 동시 설정) | 68건 |
| Geological Society of Korea 2중(426/429) | **420** (영문+원어 동시 설정) | 236건 |

- 개별 논문 저널 수동 교정 다수 (ref 2102/2103/2155/2331/2520/2672/2679/2813/2710/3610/2936/3571 등)
- 도서 3건은 `journal=None` (저널 아님, ref 2372/2376/2387 최덕근)
- **병합 후 빈 저널은 `is_deleted=True` soft delete**: 424, 438, 479, 507, 531, 532, 493, 497, 492, 485, 477, 482, 508, 566, 642

## 관련 페이지

- [KPRDB](kprdb.md) — Reference 모델, source 필드
- [북한 논문 데이터](north-korea-data.md) — Oh et al. 2023 후속 NK 논문 PDF/저자 매칭 상세
- [PDF 파이프라인](pdf-pipeline.md) — OCR 및 추출 처리
- [프로젝트 개요](overview.md) — 최신 데이터 규모 집계

---
*Sources: 029, 037, 038, 039, 040, 041, 042, 043, 044, 045, 046, 049, P17, 068 (저널 정리), docs/reference_import_report.md + reference_import_report2.md (방법론만).*
