# 055. 워크스페이스 필드 추가, 좌표 파싱 개선, 추출 파이프라인 실행

**날짜**: 2026-04-03  
**버전**: 0.3.11 → 0.3.12

---

## 1. 워크스페이스 입력 필드 추가

### 1.1 페이지 번호 자동 기록 (page_no)

세 모델 모두에 `page_no` 필드 추가:
- `ReferenceSiteCandidate.page_no` (IntegerField, nullable)
- `ReferenceTaxon.page_no` (IntegerField, nullable)
- `ReferenceTaxonSpecimen.page_no` (IntegerField, nullable)

입력 시 현재 PDF 뷰어 페이지(`currentPage`)를 자동 기록. UI에서는 숨김 처리(hidden input)하여 사용자에게 보이지 않으며 참고용으로만 DB에 저장.

### 1.2 모식표본 유형 (type_specimen)

`ReferenceTaxonSpecimen`에 `type_specimen` 필드 추가:
- choices: `모식 아님(기본)`, `완모식(holotype)`, `부모식(paratype)`, `총모식(syntype)`, `후모식(lectotype)`, `신모식(neotype)`
- 표본 목록에 badge로 표시
- 신규 입력 및 편집 시 드롭다운 선택

### 1.3 위경도 decimal 자동 변환

`ReferenceSiteCandidate`에 decimal 좌표 필드 추가:
- `latitude_decimal` (FloatField, nullable)
- `longitude_decimal` (FloatField, nullable)
- 기존 `latitude`/`longitude`는 원본 텍스트(DMS 등) 그대로 유지
- 서버 사이드 `parse_coordinate_to_decimal()` 함수로 DMS/DM/decimal 자동 변환
- 산지 카드 view에 원본 → decimal 변환 결과 표시

### 1.4 UI 용어 변경

- "산지후보" → "화석산지" (새 화석산지 폼 제목)

---

## 2. 좌표 파싱 개선 (extract_from_ocr.py)

### 2.1 방향 접두사 지원

DMS/DM/DD 패턴에서 방향 문자(N/S/E/W)가 앞에 올 수 있도록 개선:
- 기존: `36°39'14"N` (접미사만)
- 추가: `N36°39'14"` (접두사), `N36°39'14"N` (양쪽)

`_pick_direction()` 헬퍼 함수로 접두사/접미사 우선순위 처리.

### 2.2 DMS 패턴 후행 방향 제한

후행 방향 문자 뒤에 숫자가 오면 매칭하지 않도록 수정 (`[NSEW](?!\s*\d)`):
- `36°39'14"N 128°...` 에서 N이 다음 좌표의 일부로 소비되는 문제 방지

### 2.3 `parse_raw_to_lat_lng()` 함수 추가

`extract_coordinates()`는 `is_korea_coord` 필터(lat 33~43.5, lng 124~132)로 한국 범위만 반환하므로, 해외 좌표(중국 산동성 lng≈116 등)의 raw 재파싱이 실패하는 문제가 있었음.

`parse_raw_to_lat_lng(raw)`: region 필터 없이 DMS/DM/DD를 파싱하여 (lat, lng) 튜플 반환.
- `extract_from_ocr.py`에 정의, `fix_coord_precision.py`에서 임포트
- `augment_with_claude()`에서 이 함수로 Claude 좌표를 자동 교정
- 변환 전후 차이 1도 초과 시 skip (오파싱 방지 안전장치)

### 2.4 `fix_coord_precision.py` 개선

기존 파일에 남아있는 부정확한 좌표를 사후 수정하는 스크립트:
- `parse_raw_to_lat_lng()` 임포트 (중복 코드 제거)
- 1도 이상 차이 시 `[SKIP]` 처리
- 전체 실행 결과: 32건/15파일 수정, 4건 오파싱 skip
- 원인: Claude의 DMS→decimal 변환 부정확 + 이전 `toFixed(4)` 잔여

앞으로는 추출 파이프라인 자체에서 자동 교정되므로, 이 스크립트는 기존 데이터 사후 수정용으로만 사용.

---

## 3. PDF 뷰어 좌표 표시 정밀도

`pdf_detail.html`에서 좌표 표시를 `toFixed(4)` → `toFixed(6)`으로 변경:
- 추출 탭, Claude 비교 탭 모두 적용

---

## 4. PDF 다운로드/뷰어 접근 권한 변경

`reference_detail_body.html`:
- PDF 다운로드 링크와 PDF 보기 링크를 로그인 사용자에게만 표시
- 비로그인 사용자에게는 PDF 아이콘만 표시 (다운로드 불가)

`kprdb/views.py`:
- `pdf_detail`: 관리자 전용(`check_admin`) 제한 제거 → 로그인 사용자 모두 접근 가능
- `pdf_page_image`: 관리자 전용 제한 제거 → 로그인 사용자 모두 접근 가능

---

## 5. 통합 추출 파이프라인 실행

`extract_from_ocr.py --claude --claude-model sonnet --force` 실행:

| 연도 | 건수 | 좌표 | Taxa | 신종 | 표본 | Figures |
|------|------|------|------|------|------|---------|
| 2020 | 18 | 87 | 684 | 38 | 236 | 123 |
| 2019 | 16 | 8 | 485 | 28 | 165 | 94 |
| 2018 | 14 | 1 | 345 | 18 | 82 | 99 |
| 2017 | 28 | 10 | 434 | 6 | 260 | 59 |
| 2016 | 21 | 43 | 807 | 33 | 120 | 120 |
| 2015 | 20 | 12 | 760 | 25 | 263 | 107 |
| 2014 | 36 | 36 | 789 | 15 | 150 | 164 |
| 2013 | 10 | (진행중) | | | | |
| 2002 | 6 | 8 | 236 | 6 | 16 | 62 |

누적 완료: 2024~2013 + 2002 (약 170건/967건)

---

## 6. 마이그레이션

- `kprdb/migrations/0010_historicalreferencetaxon_page_no_and_more.py`
- fsis, ghdb 양쪽 DB 적용 완료

---

## 변경 파일 목록

- `kprdb/models.py` — page_no, type_specimen, latitude_decimal/longitude_decimal 필드 추가
- `kprdb/views_task.py` — 좌표 파싱 유틸, 모든 add/edit API에 신규 필드 처리
- `kprdb/templates/kprdb/task_workspace.html` — 폼/JS 업데이트, page_no 숨김, type_specimen 드롭다운
- `kprdb/templates/kprdb/_taxon_card.html` — 표본 모식표본 컬럼, page_no hidden
- `kprdb/templates/kprdb/pdf_detail.html` — 좌표 toFixed(6)
- `kprdb/templates/kprdb/reference_detail_body.html` — PDF 다운로드 로그인 제한
- `kprdb/views.py` — pdf_detail/pdf_page_image 관리자 제한 제거
- `scripts/extract_from_ocr.py` — 방향 접두사 지원, Claude 좌표 정밀도 교정
- `config/version.py` — 0.3.11 → 0.3.12
