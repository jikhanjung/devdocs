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

## 관련 페이지

- [KPRDB](kprdb.md) — Reference 모델, source 필드
- [PDF 파이프라인](pdf-pipeline.md) — OCR 및 추출 처리
- [프로젝트 개요](overview.md)

---
*Sources: 029, 037, 038, 039, 040, 041, 042, 043, 044, 045, 046, 049, P17*
