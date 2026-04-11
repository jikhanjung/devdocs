# 논문 PDF 확보 및 북한 논문 서지정보 입력 보고서

**작성일**: 2026-03-26

## 1. 배경

프로덕션 DB 기준 (2026-03-24 이전):
- 전체 논문: 869건
- PDF 보유: 326건 (37%)
- PDF 미보유: 543건

PDF가 없는 543건에 대해 자동화된 5단계 확보 전략을 수립하고 실행.

## 2. PDF 확보 5단계 전략 및 결과

### 2-1. DOI 기반 OA 확인 (82건 대상)

- **방법**: DOI → Unpaywall API로 Open Access 여부 확인 → OA PDF URL로 직접 다운로드
- **결과**: 13건 확보
  - J-STAGE (일본학술지전자저널): 12건 — `/_article/` → `/_pdf/` URL 치환
  - Unpaywall에서 paywall 판정이었지만 실제 J-STAGE에서 OA: 1건

| 상태 | 건수 |
|------|------|
| OA PDF 다운로드 성공 | 13건 |
| Paywall (Springer) | 52건 |
| Paywall (jgsk.or.kr) | 9건 |
| Paywall (기타: Elsevier, BioOne, J-STAGE) | 5건 |
| OA이지만 PDF URL 없음 | 3건 |

### 2-2. ejournal URL 직접 접근 (18건 대상)

- **방법**: jpaleodb 논문의 `ejournal_url` 필드에서 PDF 링크 추출
- **결과**: 10건 확보

| 도메인 | 건수 | 성공 | 비고 |
|--------|------|------|------|
| umdb.um.u-tokyo.ac.jp | 8건 | 8건 | 도쿄대 박물관 Kobayashi 시리즈 |
| dinosaur.pref.fukui.jp | 2건 | 2건 | 후쿠이 공룡박물관 |
| hdl.handle.net | 6건 | 0건 | 인증/404 |
| palaeo-soc-japan.jp | 1건 | 0건 | 리다이렉트 실패 |
| ci.nii.ac.jp | 1건 | 0건 | PDF 없음 |

### 2-3. 한국 저널 제목 매칭 (98건 대상)

- **방법**:
  - 한국고생물학회지: 이미 다운로드한 DBpia PDF 368건과 제목 매칭 (14건 매칭)
  - 한국지질학회지: CrossRef API로 DOI 확보 → DOI resolve → DBpia nodeId → `pdfView.do` 다운로드 (22건 매칭)
- **결과**: 36건 확보 (실패 0건)

| 저널 | 대상 | 매칭 | PDF 확보 |
|------|------|------|----------|
| 한국고생물학회지 (JKPS) | 37건 | 14건 | 14건 |
| 한국지질학회지 (JGSK) | 25건 | 22건 | 22건 |
| 기타 한국 저널 | 36건 | 0건 | 0건 |

### 2-4. 해외 저널 CrossRef 제목 검색 (243건 대상)

- **방법**: 제목으로 CrossRef API 검색 → DOI 확보 → Unpaywall OA 확인 → PDF 다운로드
- **결과**: 1건 확보

| 작업 | 건수 |
|------|------|
| CrossRef DOI 확보 | 123건 |
| OA PDF 다운로드 | 1건 (J-STAGE) |
| Paywall | 93건 |
| DOI 찾았지만 OA 아님 | 30건 |
| CrossRef에서 못 찾음 | 120건 |

### 2-5. Semantic Scholar 검색 (339건 대상)

- **방법**: Semantic Scholar API로 제목 검색 → OA PDF URL 확인
- **결과**: PDF URL 발견 1건 (다운로드 시도했으나 접근 차단)
- **비고**: Rate limit이 매우 엄격 (분당 1건 수준), 5시간 이상 소요

| 상태 | 건수 |
|------|------|
| 논문 발견 + OA PDF URL 있음 | 1건 |
| 논문 발견 + PDF 없음 | 11건 |
| 논문 미발견 | 33건 |
| 나머지 (이전 단계에서 분류) | 294건 |

### 요약

| 단계 | 대상 | PDF 확보 |
|------|------|----------|
| 1. DOI → Unpaywall | 82건 | **13건** |
| 2. ejournal URL | 18건 | **10건** |
| 3. 한국 저널 DBpia | 98건 | **36건** |
| 4. 해외 CrossRef | 243건 | **1건** |
| 5. Semantic Scholar | 339건 | **0건** |
| **합계** | | **60건** |

기존 326건 + 60건 = **386건** (기존 869건 중 44%)

### 미확보 논문 분류 (483건)

| 상태 | 건수 | 설명 |
|------|------|------|
| no_lead | 189건 | 저널 미상, DOI/URL 없음 |
| crossref_not_found | 102건 | CrossRef에서 DOI 못 찾음 |
| paywall_springer | 52건 | Springer 구독 필요 |
| ss_not_found | 33건 | Semantic Scholar에서도 못 찾음 |
| doi_found_no_oa | 30건 | DOI 있지만 OA 아님 |
| paywall (기타) | ~60건 | Cambridge, T&F, Elsevier 등 |
| 기타 | ~17건 | ejournal 실패, 제목 짧음 등 |

미확보 논문의 PDF 확보를 위해서는 기관 구독, 저자 직접 요청, 도서관 방문 등이 필요.

## 3. 북한 논문 서지정보 입력

### 출처

Oh et al. (2023) 논문의 참고문헌 목록에서 북한 고생물학 논문 61건 추출.

### 입력 내용

- **61건** 서지정보 (영문 제목, 저자, 연도, 저널, 권호, 페이지)
- 출처: `Oh et al. (2023)`
- 한글 인용형은 `remarks` 필드에 보존
- 저자 47명 신규 생성, 79회 기존 저자 재사용
- PDF 없음 (북한 학술지로 확보 불가)

### 저널 분포

| 저널 | 건수 |
|------|------|
| Journal of Kim Il Sung University (Natural Science) | 28건 |
| Bulletin of the Academy of Sciences of the D.P.R. Korea | 7건 |
| Geology Science | 5건 |
| Geology and Geography | 5건 |
| Geological Survey | 5건 |
| Journal of Kim Il Sung University (Earth Science and Geology) | 2건 |
| 기타 (Kim Chaek Univ., 일본 저널, 서적) | 9건 |

### 주요 연구 분야

- 코노돈트 생층서 (Won, Hong, Pae, Cha 등)
- 삼엽충 (Kim D.S.)
- 선캄브리아기 미화석/아크리타크 (Kim & Li, Ri & Pak)
- 에디아카라 화석 (Jang, Kang, Ri)
- 완족류/산호 (Pak Y.S.)
- 식물화석 (Kang J.G.)
- 생흔화석 (So, Won)

### 서지정보 파일

- `dbpia_jkps/north_korea_paleontology_refs.json` — 61건 (영문/한글 인용형 매핑 포함)
