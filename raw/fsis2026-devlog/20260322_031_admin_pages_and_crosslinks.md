# 031. 관리자 페이지 추가, 화석산지-논문 상호 링크, UI 개선 (2026-03-22)

## 관리자 전용 페이지

### 추출 표본번호 페이지 (`/kprdb/extracted_specimens/`)
- PDF에서 자동 추출된 표본번호 664건 일괄 조회
- FSIS MuseumSpecimen + KPRDB ReferenceTaxonSpecimen 매칭 표시
- 기관별 필터 (KOPRIF 445, JBNU 90, type_specimen 53 등)
- 매칭/미매칭 필터

### KPRDB 표본 목록 페이지 (`/kprdb/taxon_specimens/`)
- ReferenceTaxonSpecimen 467건 전체 목록
- FSIS MuseumSpecimen과 표본번호 매칭 (현재 2건 매칭)
- 검색 (표본번호, 위치, 논문 제목)
- 매칭 필터를 DB 쿼리 레벨로 구현 (페이지 단위가 아닌 전체 대상)

## 화석산지 ↔ 논문 상호 링크

### fossil_site_detail → 논문 목록
- 연결된 논문을 연도 내림차순, 저자명 오름차순으로 표시
- 논문 제목에 reference_detail 링크

### reference_detail → 화석산지 목록
- 연결된 화석산지를 시도, 시군구 순으로 표시
- 산지코드에 fossil_site_detail 링크

## ghdb 버그 수정

### 일괄 입력 리다이렉트 URL
- `/ghdb/` → `/` 로 수정 (ghdb는 별도 컨테이너에서 루트 URL이 `/`)

### INSTALLED_APPS 에러
- `FossilSiteReference → kprdb.Reference` FK 때문에 ghdb 컨테이너 기동 실패
- `config/settings_ghdb.py`에 `kprdb.apps.KprdbConfig` 추가

## UI 개선

### 테이블 링크 색상
- `geo-theme.css`의 `.geo-table tbody td a` 색상 변경
- `var(--geo-stone)` (일반 텍스트와 동일) → `#2c6faa` (파란색)
- hover 시 밑줄 추가

### 시스템관리 메뉴 추가
- PDF 처리현황
- 추출 표본번호
- KPRDB 표본목록

## 버전 이력
| 버전 | 내용 |
|------|------|
| 0.2.24 | fossil_site_detail 논문 목록 |
| 0.2.25 | reference_detail 화석산지 목록 |
| 0.2.26 | 추출 표본번호 페이지 |
| 0.2.27 | KPRDB 표본목록 페이지 |
| 0.2.28 | 표본목록 매칭 필터 수정 (DB 레벨) |
| 0.2.29 | ghdb 일괄입력 리다이렉트 수정 |
| 0.2.30 | ghdb INSTALLED_APPS에 kprdb 추가 |
| 0.2.31 | 테이블 링크 색상 파란색으로 변경 |
