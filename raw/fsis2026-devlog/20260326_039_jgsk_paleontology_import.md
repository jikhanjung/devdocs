# 039. 한국지질학회지 고생물 논문 검색 및 임포트

**날짜**: 2026-03-26
**버전**: 0.2.74

## 개요

CrossRef API로 한국지질학회지(ISSN 0435-4036) 고생물 관련 논문을 검색하고,
DOI → DBpia nodeId 매핑을 통해 PDF를 다운로드하여 프로덕션 DB에 임포트.

## 1. CrossRef 검색

- ISSN `0435-4036`로 고생물 관련 키워드 35종 검색
- 초기 검색 결과: **312건** (1966~2026)
- 고생물 키워드 필터링 후: **140건**

### 필터링 키워드
fossil, paleontology, trilobite, brachiopod, dinosaur, conodont, graptolite,
fusulinid, foraminifer, ichnofossil, trace fossil, footprint, trackway,
pollen, spore, ostracod, radiolaria, biostratigraphy, fauna, flora,
bivalve, coral, diatom, nannofossil, stromatolite, microbial, bird,
charophyte, Coptoclava, Kainella, Lockeia, paleoclimate, paleoenvironment 등

### 제외된 208건
주로 층서학, 퇴적학, 화성암학, 구조지질학, 고자기학, 지구화학 논문

## 2. DOI → PDF 경로 확인

DOI 리졸브 결과 두 가지 경로로 분기:

| 경로 | 건수 | PDF 확보 |
|------|------|----------|
| DBpia (`pdfView.do?nodeId=`) | 126건 | 전부 성공 |
| jgsk.or.kr (자체 사이트) | 14건 | PDF 미확보 (2013~2017 최신) |

- DBpia PDF URL: `https://www.dbpia.co.kr/pdf/pdfView.do?nodeId={nodeId}`
- DOI → DBpia URL에서 `nodeId` 파라미터 추출

## 3. PDF 다운로드

- **126건 전부 성공** (656MB), 실패 0건
- 저장 위치: `dbpia_jkps/jgsk_pdfs/{nodeId}.pdf`
- 메타데이터: `dbpia_jkps/jgsk_paleontology_filtered.json` (140건, DOI/nodeId 포함)

## 4. DB 임포트 결과

| 작업 | 건수 |
|------|------|
| 이미 PDF 있음 (스킵) | 29건 |
| 기존 레코드에 PDF 연결 | 22건 |
| 신규 Reference 생성 | 88건 |
| 저자 신규 생성 | 137명 |
| 저자 재사용 | 111회 |

- 신규 논문 source: `crossref_jgsk`
- UID 패턴: `kprdb:bib:jgsk:sha256:{hash}`

## 5. 프로덕션 DB 현황 (임포트 후)

| 구분 | 건수 |
|------|------|
| 전체 논문 | 1,301건 |
| PDF 보유 | 755건 (58%) |
| 출처: 기존 | 869건 |
| 출처: dbpia (고생물학회지) | 169건 |
| 출처: jpaleodb | 175건 |
| 출처: crossref_jgsk (지질학회지) | 88건 |

## 6. 미확보 PDF (14건, jgsk.or.kr)

jgsk.or.kr 자체 사이트에서 PDF 다운로드 경로를 찾지 못한 논문 (2013~2017):

| 연도 | 제목 |
|------|------|
| 2013 | Interpretation of paleo-environment and origin of fractures... |
| 2013 | Preliminary research on the aquatic coleopteran, Coptoclava... |
| 2013 | Paleoclimatic investigation using trace elemental compositions... |
| 2014 | A case study on the preservation of dinosaur fossil sites... |
| 2014 | Calcareous nannofossil biostratigraphy of North Pond area... |
| 2014 | Roll-up clasts in the Cretaceous Haman Formation... |
| 2014 | Carbon isotopic composition of Miocene-Pliocene terrestrial plant... |
| 2015 | Study history and research ethics of the dinosaur, pterosaur and bird... |
| 2015 | Unique burrows in the Cretaceous Hasandong Formation... |
| 2015 | Soft-sediment deformation structures in the Cretaceous Gyeokpori... |
| 2016 | A review of the middle Darriwilian carbon isotope excursion... |
| 2016 | An assessment of geosites in the Cretaceous Wido Volcanics |
| 2017 | Importance of the paleontological resources for the stratigraphy... |
| 2017 | The stratigraphy and correlation of the upper Paleozoic Pyeongan... |

## 출력 파일

- `dbpia_jkps/jgsk_paleontology_articles.json` — CrossRef 검색 전체 312건
- `dbpia_jkps/jgsk_paleontology_filtered.json` — 고생물 필터 140건 (nodeId, PDF 다운로드 여부 포함)
- `dbpia_jkps/jgsk_pdfs/{nodeId}.pdf` — PDF 126건 (프로덕션 DB 임포트 후 삭제, `.gitignore` 제외)
