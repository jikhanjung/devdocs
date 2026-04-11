# 북한 논문 데이터 (NK Paleontology Corpus)

북한 고생물학 논문은 남한·해외 학술지와 유통 경로가 다르고, 저자 이름이 영문 이니셜로만 들어오는 등 별도 처리 전략이 필요한 [KPRDB](kprdb.md)의 하위 코퍼스다. 현재 **101건** 등록, 그중 **96건**에 PDF 연결. 주요 출처는 **Oh et al. (2023)** 논문의 인용 문헌이며, PDF는 `nkpaleo/` 디렉토리에서 수동 매칭.

이 페이지는 2026-03-26 ~ 2026-03-27 세션에서 수행된 북한 논문 통합 작업과 그 후속 보고서들을 정리한다.

## 1. 선행 연구

**Oh, Y. et al. (2023)**. *Reappraisal of the Neoproterozoic to middle Paleozoic fossils of North Korea and its tectonic implication*. *Geosciences Journal* 27(6): 661–687.

이 논문이 북한 고생물학 논문의 재정리 및 인용 체계의 기준점. KPRDB에서는 이 논문에서 인용된 61건을 초기 seed로 `source='YJOh'` 태그와 함께 등록.

## 2. PDF 매칭 현황 (2026-03-27 기준)

| 항목 | 건수 |
|---|---|
| **전체 북한 논문** | 101 |
| **PDF 연결됨** | 96 |
| **PDF 미확보** | 5 |

### PDF 미확보 목록 (`docs/pdf_missing_nk.md`)

| ID | 연도 | 서지사항 |
|---|---|---|
| 3637 | 1964 | 조선 동북부와 쏘련 연해주 남부의 지질 구성과 지하 자원 |
| 3535 | 2011 | Kim M.H., Rim T.S. — *On the Stromatoporoidea fossil newly discovered in upper member of Mandal Formation in the Sungho area*. Geologic and Geographic Science 1: 17–18 |
| 3630 | 2022 | Han, Geum-Chul, Choi, Rye-Sun — *The Gomphotheridae from Myongchon area*. Journal of Kim Il Sung University (Earth Science and Geology) 68(1): 109–111 |

추가로 **docs/north_korea_pdf_matching.md**에는 PDF 연결된 96건의 전체 목록(ID·연도·저자·한글 제목·PDF 파일명)이 있다. 1964년부터 2022년까지 분포. 대부분 `{ref_id}.pdf` 파일명 규칙.

## 3. 저자 한글 이름 매핑

**문제**: 북한 논문 저자들이 DB에 영문 이니셜 형식(예: `Won C.G.`)으로만 입력되어 있었음. 원본 논문의 한글 인용형(예: `원철국(1997:11)`)과 대조하여 한글 이름을 매칭해야 검색 품질이 올라감.

**결과**: 51명 중 38명 매핑 완료 (`docs/north_korea_author_name_mapping.md`).

### 확인된 매칭 (34명, 한글 인용형에서 직접 확인)

| 영문 | 한글 | 출처 확인 예 |
|---|---|---|
| Cha M.D. | 차명도 | 차명도(2011:6) |
| Cho S.B. | 조성복 | 원철국조성복(2004:1) |
| Ha J.C. | 하종철 | 홍성권 원철국 하종철(1994:1) |
| Hong S.G. | 홍성권 | 홍성권 리정임(1990:1) 등 다수 |
| Jang D.S. | 장덕성 | 장덕성 서광식(2006:4) |
| Jang I.N. | 장일남 | 장일남 리현수 강진건(1994:8) |
| Jang T.S. | 장덕성 | 장덕성 김철근(2004:6) — 이니셜 T.S.로 표기 |
| Jong C.B. | 정창복 | 정창복(1972:5) |
| Jong H.S. | 정호상 | 정호상 홍성권 원철국(2001:6) |
| Kang J.C. | 강준철 | 강준철 장덕성(2011:8) |
| Kang J.G. | 강진건 | 강진건(2012:5) |
| Kim B.S. | 김병성 | 김병성(2013:3) |
| Kim C.G. | 김철근 | 장덕성 김철근(2004:6) |
| Kim C.S. | 김철수 | 김철수 리철준 서광식(2008:7) |
| Kim D.S. | 김덕성 | 김덕성(1973:5), (1977:4), (1984:4), (1984:5) |
| Kim H.S. | 김혜성 | 서광식 원철국 김혜성(2014:11) |
| Kim K.R. | 김광룡 | 김광룡(2011:9) |
| Kim K.Y. | 김기영 | 김기영(1972:2) |
| Kim M.H. | 김명학 | 김명학 림동수(2011) |
| Li S.D. | 리상도 | 김덕성 리상도(1989:6) |
| Pae P.H. | 배봉하 | 장덕성 배봉하(2007:9) |
| Pae Y.J. | 배용준 | 배용준 홍성권 원철국(1991:8) |
| Pak Y.C. | 박영철 | 박영철 리현수(2003:6) |
| Pak Y.S. | 박용선 | 박용선(1966:2), (1967:1), (1976:2) |
| Ri C.J. | 리철준 | 리철준 김철수 김철근 서광식(2008:9) |
| Ri H.S. | 리현수 | 박영철 리현수(2003:6) |
| Ri J.Y. | 리정임 | 홍성권 리정임(1990:1) |
| Ri S.D. | 리상도 | 리상도 박영철(1994:5), (1999:1) |
| Ri S.G. | 리선국 | 리선국(2011:10) |
| Ri U.B. | 리은빛 | 리은빛(2019:6) |
| So K.S. | 서광식 | 서광식(2010:4), 서광식 원철국 김혜성(2014:11) |
| Song J.H. | 송진혁 | 원철국 송진혁(2021:1) |
| Won C.G. | 원철국 | 원철국(1997:11) 등 다수 |

### 추정 매칭 (5명)

| 영문 | 한글 (추정) | 근거 |
|---|---|---|
| Choe J.H. | 최종학 | Kim et al. (1987) 공저자, Choe = 최 |
| Pak M.H. | 박명학 | Pak et al. (2009) 제1저자 |
| Ri S.B. | 리상보 | Pak et al. (2009) 공저자 |
| Sin M.G. | 신명근 | Kim et al. (1987) 공저자 |
| Rim T.S. | 림동수 | 김명학 림동수(2011) = Kim and Rim (2011) |

### 미매칭 (13명)

주로:
- 한글 인용형이 없는 공저자
- 중국인/일본인 연구자 (Peng P., Yabe H., Sugiyama T., Zhai M.G., Zhang Z.Y., Yang J.H. 등)
- 동명이인 다수로 특정 불가 (Kim Y. 등)

### 이니셜 불일치 사례

**장덕성**: `Jang T.S.` 와 `Jang D.S.` 두 가지로 표기
- Jang T.S.: Jang and Kim (2004), Jang et al. (2005), Jang and Pae (2007)
- Jang D.S.: Jang and So (2006)
- → **동일 인물** (장덕성 김철근, 장덕성 배봉하, 장덕성 서광식에서 확인)

**리상도**: `Li S.D.` 와 `Ri S.D.` 두 가지로 표기
- Li S.D.: Kim and Li (1989), (1993), (1995) — 김덕성 리상도
- Ri S.D.: Ri and Pak (1994), (1999), (2000) — 리상도 박영철
- → **동일 인물** (이름의 로마자 표기 차이, 한국어 `리` vs 중국어 pinyin `Li`)

## 4. PDF 확보 프로세스

북한 논문 96건의 PDF는 **`nkpaleo/` 디렉토리**에서 한국어 파일명 매칭으로 임포트:

- 55건: 기존 DB 레코드에 PDF 연결
- 32건: 신규 레코드 생성 (PDF-only)
- 9건: 1:N 매칭 해결 (한 PDF가 여러 문헌 후보)
- 23명 신규 저자 생성

도서 4건 별도 처리:
- 조선의화석 1 (고생대편)
- 조선의화석 2 (중생대편)
- 조선고생물화석
- 조선의 중생대 신의주 생물군

## 5. 주요 연구자 분포 (한글 매칭 기준)

매칭된 저자를 발표 시기와 주제로 그룹화:

| 세대 / 시기 | 주요 저자 | 주 연구 분야 |
|---|---|---|
| 1960s–70s | 박용선, 김덕성, 정창복, 김기영 | 캄브리아기 삼엽충, 송림 력암층 완족류 |
| 1980s | 김덕성, 박용선, 강진건, 김영길 | 데본기 식물/완족류, 평양지구 삼엽충 |
| 1990s | 홍성권, 박영철, 리상도, 원철국, 배봉하 | 오르도비스 코노돈트, 상원계 acritarch |
| 2000s | 원철국, 김덕성, 장덕성, 서광식, 배봉하 | 만달주층, 묵천통, 평산/연탄 신원생대 |
| 2010s | 강준철, 강진건, 김명학, 리철준, 원철국 | 에디아카라기, 림진군층 완족류, 씰루르기 |
| 2020s | 원철국, 서광식, 리철준 | 캄브리아기 부추치류, 백토동화석보호구 |

## 6. 연관 시스템

### KPRDB
- 모든 레코드 `source='YJOh'` 또는 nkpaleo 매칭
- [KPRDB](kprdb.md)의 저자/저널/논문 모델을 그대로 사용 (별도 앱이 아님)
- 저널은 주로 `조선지질 · 지질과학 · 김일성종합대학학보 (지구과학 및 지질)` 등

### 데이터 수집 맥락
[데이터 수집](data-import.md)의 "북한 논문 임포트" 섹션이 초기 잡 기록이고, 이 페이지가 그 후속 정리 보고서다. reference import 전체 집계(1,354건, 756 PDF)는 `data-import.md`의 [Import 보고서 집계](data-import.md#import-보고서-집계-2026-03-26-기준) 참조.

### PaperMeister와의 관계
흥미롭게도 같은 devdocs 위키 내의 [PaperMeister](papermeister-overview.md)는 동일한 문제 — "여러 source에서 온 문헌을 canonical corpus로 만들기" — 를 **일반적인 플랫폼**으로 해결하려는 제품이다. KPRDB의 북한 논문 처리는 이런 corpus 통합 작업의 **구체적인 도메인 사례**로 볼 수 있다.

## 7. 향후 과제

- **13명 미매칭 저자** 추가 조사 (특히 Pak et al. 2009 계열 공저자)
- **저자 정규화 DB 테이블**: 현재는 ad-hoc 매핑. 향후 `Author.alternate_names` 필드 또는 별도 alias 테이블로 통합 가능
- **PDF 3건 미확보** 확보 시도 (김일성종합대학학보 2022년 건 포함)
- **일본어/중국어 공저자** 로마자 표기 정규화

## 관련 페이지

- [프로젝트 개요](fsis2026-overview.md) — KOFHIN 전체 맥락
- [KPRDB](kprdb.md) — Reference/Author/Journal 모델
- [데이터 수집](data-import.md) — 전체 reference import 파이프라인
- [KOFHIN 매뉴얼](kofhin-manuals.md) — 논문/저자 입력 UI
- [PaperMeister 개요](papermeister-overview.md) — 같은 종류의 corpus 통합 문제를 일반 플랫폼으로 해결

---
*Sources: docs/north_korea_pdf_matching.md (2026-03-27), docs/north_korea_author_name_mapping.md (2026-03-27), docs/pdf_missing_nk.md (2026-04-07), docs/reference_import_report.md 북한 섹션.*
