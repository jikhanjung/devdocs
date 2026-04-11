# 한국 화석 관련 논문 데이터 수집 및 임포트 보고서

**작성일**: 2026-03-26
**프로젝트**: KPRDB (한반도 고생물 데이터베이스)

## 1. 개요

한반도 화석 관련 논문의 서지정보와 PDF 원문을 체계적으로 확보하기 위해 4개 출처에서 데이터를 수집하였다. 수집 대상은 jpaleodb(일본 고생물학 데이터베이스), 한국고생물학회지(DBpia), 한국지질학회지(CrossRef/DBpia), Geosciences Journal(CrossRef)이며, 각 출처별로 웹 스크래핑, API 호출, PDF 자동 다운로드를 수행하여 KPRDB에 임포트하였다.

### 최종 결과 요약

| 출처 | 수집 논문 | 신규 임포트 | PDF 확보 | 비고 |
|------|-----------|-------------|----------|------|
| jpaleodb | 177건 | 175건 | 30건 | 표본 1,532건 별도 수집 |
| 한국고생물학회지 (DBpia) | 412건 | 197건 | 368건 | 137건 기존 레코드에 PDF 연결 |
| 한국지질학회지 (CrossRef) | 140건 | 88건 | 126건 | 14건 jgsk.or.kr PDF 미확보 |
| Geosciences Journal (CrossRef) | 80건 | 53건 | 1건 | Springer 구독 필요 |
| **합계** | **809건** | **513건** | **525건** | |

- DB 총 논문: **1,354건**, PDF 보유: **756건 (56%)**
- 중복 논문: 5그룹 확인 (미병합)

---

## 2. 출처별 수집 과정

### 2.1 jpaleodb (일본 고생물학 데이터베이스)

#### 배경

jpaleodb(http://www.jpaleodb.jp)는 일본 고생물학회가 운영하는 데이터베이스로, 한반도 관련 논문과 표본 정보를 다수 보유하고 있다. 특히 일제강점기 및 초기 한국 고생물학 논문에 대한 서지정보가 풍부하다.

#### 수집 방법

1. **논문 검색**: 웹사이트에서 "Korea" 및 "Chosen"(조선) 키워드로 논문 검색
2. **결과 저장**: 검색 결과를 JSON으로 저장 (177건, UID 기반 중복 제거 후 175건)
3. **표본 수집**: 각 논문의 표본 목록 페이지에서 표본 정보 스크래핑 (1,532건)

#### 수집 데이터

- 논문 필드: 저자, 연도, 제목, 영문/일문 서지사항, refid, 표본목록 URL, ejournal URL, DOI
- 표본 필드: 기재 종명, 표본 유형, 분류 계통, 산지, 지층, 지질시대, 소장 기관, 등록번호

#### DB 임포트

`import_jpaleodb` management command를 통해 175건 임포트:
- UID: `jpaleodb:ref:{refid}` 형식
- 저자 파싱: `"KOBAYASHI, T., HUKASAWA, T."` → 개별 Author 객체 (기존 DB와 정규화 비교 후 매칭/생성)
- 저널: `title_e` 정확 매칭 → `get_or_create`
- `remarks`에 jpaleodb 출처 정보 기록 (refid, specimen_list_url, ejournal_url)
- `source='jpaleodb'` 설정

표본 1,532건은 `SpecimenCandidate` 모델에 별도 저장 (reference_uid 문자열 매칭, FK 아님).

#### PDF 확보

`download_jpaleodb_pdfs` command로 논문의 `ejournal_url`에서 PDF 자동 다운로드:

| 도메인 | 대상 | 방법 | 성공 |
|--------|------|------|------|
| J-STAGE (jstage.jst.go.jp) | 30건 | `/_article/` → `/_pdf/` URL 치환 | 30건 |
| handle.net | 9건 | 랜딩 페이지 스크래핑 | 2건 |

- 결과: **30건 PDF 확보** (나머지는 기관 인증 필요)

---

### 2.2 한국고생물학회지 (DBpia)

#### 배경

한국고생물학회지(The Journal of the Paleontological Society of Korea, 1985~2014)는 한국 고생물학 분야의 핵심 학술지로, DBpia(https://www.dbpia.co.kr)에서 전문(全文)을 제공한다.

#### 수집 방법

DBpia 내부 API를 역분석하여 자동 수집:

1. **호 목록 조회**: `POST /voisListAjax` — publicationId `PLCT00001598`로 전체 55호 목록 확보
2. **논문 목록 조회**: `POST /journal/voisNodeList` — 각 호의 논문 목록 (nodeId, 제목, 저자, 연도)
3. **PDF 다운로드**: `GET /pdf/pdfView.do?nodeId={nodeId}` — 로그인 없이 직접 다운로드

#### 수집 결과

| 항목 | 건수 |
|------|------|
| 전체 호 | 55호 (1985~2014) |
| 전체 논문 | 412건 |
| PDF 다운로드 성공 | 368건 (2.4GB) |
| 실패 | 44건 (섹션 구분자, 실제 논문 아님) |

실패 44건은 모두 "논문", "초록", "단보", "컬럼", "서평" 등 섹션 헤더 항목으로, 실제 논문 PDF는 100% 다운로드에 성공하였다.

#### DB 임포트

`import_dbpia_jkps` management command로 다운로드된 368건과 기존 DB를 매칭:

| 처리 | 건수 |
|------|------|
| 기존 레코드에 PDF 연결 | 137건 |
| 신규 Reference 생성 | 197건 |
| 이미 PDF 있음 (스킵) | 34건 |

- 신규 논문 `source='dbpia'`
- 기존 DB에 이미 등록된 논문 중 PDF가 없었던 137건에 대해 PDF 파일을 연결

#### 데이터 정제

임포트된 항목 중 실제 논문이 아닌 것들(투고 규정, 공지사항, 편집후기 등)이 포함되어 있었다. 이들은 주로 저자가 "편집부"로 되어 있었으며, 확인 후 DB에서 삭제하였다.

---

### 2.3 한국지질학회지 (CrossRef + DBpia)

#### 배경

한국지질학회지(Journal of the Geological Society of Korea, ISSN 0435-4036)는 지질학 전반을 다루는 학술지로, 고생물학 관련 논문이 다수 포함되어 있다. CrossRef API로 고생물 관련 논문을 선별하고, DOI를 통해 DBpia에서 PDF를 확보하였다.

#### 수집 방법

1. **CrossRef API 검색**: ISSN `0435-4036`로 고생물 관련 키워드 35종 검색
2. **고생물 필터링**: 검색 결과 312건 → 키워드 기반 필터링 → 140건 선별
3. **DOI 리졸브**: DOI 접근 시 두 가지 경로로 분기
   - DBpia (`pdfView.do?nodeId=`): 126건 → PDF 전부 다운로드 성공
   - jgsk.or.kr (자체 사이트): 14건 → PDF 미확보

#### 필터링 키워드

fossil, paleontology, trilobite, brachiopod, dinosaur, conodont, graptolite, fusulinid, foraminifer, ichnofossil, trace fossil, footprint, trackway, pollen, spore, ostracod, radiolaria, biostratigraphy, fauna, flora, bivalve, coral, diatom, nannofossil, stromatolite, microbial, bird, charophyte, Coptoclava, Kainella, Lockeia, paleoclimate, paleoenvironment 등

필터링으로 제외된 172건은 주로 층서학, 퇴적학, 화성암학, 구조지질학, 고자기학, 지구화학 논문이다.

#### 수집 결과

| 항목 | 건수 |
|------|------|
| CrossRef 검색 | 312건 |
| 고생물 필터링 후 | 140건 |
| DBpia PDF 다운로드 | 126건 (656MB) |
| jgsk.or.kr PDF 미확보 | 14건 (2013~2017) |

#### DB 임포트

| 처리 | 건수 |
|------|------|
| 이미 PDF 있음 (스킵) | 29건 |
| 기존 레코드에 PDF 연결 | 22건 |
| 신규 Reference 생성 | 88건 |
| 저자 신규 생성 | 137명 |
| 저자 재사용 | 111회 |

- 신규 논문 `source='crossref_jgsk'`
- UID: `kprdb:bib:jgsk:sha256:{hash}`

---

### 2.4 Geosciences Journal (CrossRef)

#### 배경

Geosciences Journal(ISSN 1226-4806/1598-7477)은 한국지질과학연합회가 발행하고 Springer가 배포하는 영문 학술지로, 한반도 및 동아시아 고생물학 논문을 포함하고 있다. 구독 저널이라 PDF 대량 확보에 제약이 있다.

#### 수집 방법

1. **CrossRef API 검색**: 고생물 관련 키워드로 검색 → 208건
2. **고생물 필터링**: 키워드 기반 → 80건 선별
3. **PDF 확보 시도**: Unpaywall API로 Open Access 여부 확인

#### 수집 결과

| 항목 | 건수 |
|------|------|
| CrossRef 검색 | 208건 |
| 고생물 필터링 후 | 80건 |
| Springer 구독 필요 | 77건 |
| Open Access | 3건 |
| PDF 다운로드 성공 | 1건 (87KB) |

DOI 리졸브 시 `link.springer.com`으로 리다이렉트되며, 기관 구독 없이는 PDF 접근이 불가하다.

#### DB 임포트

| 처리 | 건수 |
|------|------|
| 이미 PDF 있음 (스킵) | 18건 |
| 기존 레코드 PDF 연결 대상 | 9건 (OA PDF 없어 미연결) |
| 신규 Reference 생성 | 53건 |
| 저자 신규 생성 | 134명 |
| 저자 재사용 | 47회 |

- 신규 논문 `source='crossref_gj'`
- UID: `kprdb:bib:gj:sha256:{hash}`

---

## 3. 최종 DB 현황

### 출처별 논문 수

| 출처 | 논문 수 | 비율 |
|------|---------|------|
| 기존 (수동 입력) | 869건 | 64.2% |
| jpaleodb | 175건 | 12.9% |
| 한국고생물학회지 (DBpia) | 169건 | 12.5% |
| 한국지질학회지 (CrossRef) | 88건 | 6.5% |
| Geosciences Journal (CrossRef) | 53건 | 3.9% |
| **합계** | **1,354건** | **100%** |

### PDF 확보 현황

| 구분 | 건수 | 비율 |
|------|------|------|
| PDF 보유 | 756건 | 55.8% |
| PDF 미보유 | 598건 | 44.2% |
| **합계** | **1,354건** | **100%** |

---

## 4. 기술 구현

### 관련 management commands

| 커맨드 | 기능 |
|--------|------|
| `import_jpaleodb` | jpaleodb JSON → Reference/Author/Journal 임포트 |
| `download_jpaleodb_pdfs` | jpaleodb ejournal_url에서 PDF 다운로드 (J-STAGE, handle.net) |
| `import_dbpia_jkps` | DBpia 메타데이터 + PDF → Reference 매칭/생성 |
| `download_dbpia_pdfs` | DBpia nodeId 기반 PDF 다운로드 |
| `generate_uids` | Reference UID 일괄 생성 (SCODA Schema 기반) |

### UID 체계

각 출처별로 UID 접두사를 구분하여 논문 식별:

| 출처 | UID 패턴 | 예시 |
|------|----------|------|
| jpaleodb | `jpaleodb:ref:{refid}` | `jpaleodb:ref:1234` |
| DOI 보유 | `kprdb:bib:doi:{normalized_doi}` | `kprdb:bib:doi:10.1234/...` |
| DOI 미보유 | `kprdb:bib:fp_v1:sha256:{hash}` | (서지 fingerprint 해시) |
| 지질학회지 신규 | `kprdb:bib:jgsk:sha256:{hash}` | |
| Geosciences J. 신규 | `kprdb:bib:gj:sha256:{hash}` | |

### 논문 목록 출처 필터

논문 목록 페이지에서 출처별 필터 기능 제공:
- "jpaleodb 제외" (기본값) — 기존 운영 흐름 유지
- "전체 (jpaleodb 포함)" — 모든 논문 표시
- "jpaleodb만" — jpaleodb 논문만 표시

---

## 5. 중간 산출물

### 수집 데이터 파일

| 파일 | 내용 |
|------|------|
| `jpaleodb/jpaleodb_korea_or_chosen_references.json` | jpaleodb 논문 177건 |
| `jpaleodb/jpaleodb_korea_or_chosen_specimens.json` | jpaleodb 표본 1,532건 |
| `jpaleodb/kobayashi_teiichi_references.json` | 小林貞一 논문 별도 수집 |
| `dbpia_jkps/jkps_articles.json` | 한국고생물학회지 412건 메타데이터 |
| `dbpia_jkps/jgsk_paleontology_filtered.json` | 지질학회지 고생물 140건 |
| `dbpia_jkps/geosciences_journal_filtered.json` | Geosciences J. 고생물 80건 |

PDF 파일은 프로덕션 DB 임포트 후 로컬에서 삭제 (`.gitignore` 제외).

---

## 6. 미해결 사항 및 향후 과제

### 중복 논문 (5그룹, 미병합)

| 연도 | 저자 | 기존 ID | 중복 ID | 비고 |
|------|------|---------|---------|------|
| 1998 | Yi S., Yun H., Yoon S. | 2563 | 3164 [jpaleodb] | jpaleodb에 PDF |
| 2016 | Choi B.D., Huh M. | 2637 | 2897 | 기존끼리 중복 |
| 1999 | Kim J.Y., Lee H.N., Cheong C.H. | 2221 | 3165 [jpaleodb] | jpaleodb에 PDF |
| 1999 | Yun C.S. | 2225 | 3167 [jpaleodb] | jpaleodb에 PDF |
| 1999 | Yun C.S. | 2226 | 3166 [jpaleodb] | jpaleodb에 PDF |

→ jpaleodb PDF를 기존 레코드로 이동 후 jpaleodb 레코드 삭제 예정

### PDF 미확보 논문

| 대상 | 건수 | 확보 방안 |
|------|------|-----------|
| jgsk.or.kr (지질학회지 자체사이트) | 14건 | 사이트 PDF 경로 재조사 |
| Geosciences Journal (Springer) | 77건 | 기관 구독 프록시 또는 저자 요청 |
| 기타 PDF 미보유 | ~500건 | Google Scholar, 저자 요청, 도서관 연계 |

### 향후 확장

- OCR 실패 3건 재처리
- PDF 추출 좌표/표본번호 패턴 확대
- 추가 저널 탐색 (Island Arc, Acta Geologica Sinica 등)
