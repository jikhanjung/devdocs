# 040. Geosciences Journal 고생물 논문 검색 및 임포트

**날짜**: 2026-03-26
**버전**: 0.2.74

## 개요

CrossRef API로 Geosciences Journal (ISSN 1226-4806/1598-7477) 고생물 관련 논문을 검색하여 프로덕션 DB에 임포트. Springer 구독 저널이라 PDF 확보는 제한적.

## 1. CrossRef 검색

- 고생물 관련 키워드 검색: 초기 208건 → 필터링 후 **80건** (1998~2025)
- 주요 분야: 삼엽충, 코노돈트, 공룡, 화석식물, 완족류/산호, 미화석, 생층서
- 일부 해외 연구 포함 (인도, 말레이시아, 아프리카 등)

## 2. PDF 확보

- 전체 80건 중 Springer 구독 필요: **77건**
- Open Access: **3건** (Unpaywall 확인)
- OA PDF URL 있음: 2건 → 실제 다운로드 성공: **1건** (87KB)
- DOI → `link.springer.com` 리다이렉트, 구독 없이는 PDF 접근 불가

## 3. DB 임포트 결과

| 작업 | 건수 |
|------|------|
| 이미 PDF 있음 (스킵) | 18건 |
| 기존 레코드 PDF 연결 대상 | 9건 (OA PDF 없어 미연결) |
| 신규 Reference 생성 | 53건 |
| 저자 신규 생성 | 134명 |
| 저자 재사용 | 47회 |

- 신규 논문 source: `crossref_gj`
- UID 패턴: `kprdb:bib:gj:sha256:{hash}`

## 4. 프로덕션 DB 현황 (임포트 후)

| 구분 | 건수 |
|------|------|
| 전체 논문 | 1,354건 |
| PDF 보유 | 756건 (56%) |
| 출처: 기존 | 869건 |
| 출처: dbpia (고생물학회지) | 169건 |
| 출처: jpaleodb | 175건 |
| 출처: crossref_jgsk (지질학회지) | 88건 |
| 출처: crossref_gj (Geosciences J.) | 53건 |

## 출력 파일

- `dbpia_jkps/geosciences_journal_filtered.json` — 고생물 필터 80건 (DOI, OA PDF URL 포함)

## 비고

- Geosciences Journal은 Springer 구독 저널이라 PDF 대량 확보 어려움
- 기관 구독 프록시 또는 저자 직접 요청으로 개별 확보 필요
- 향후 대학 도서관 연계 시 일괄 확보 가능
