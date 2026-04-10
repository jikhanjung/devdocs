# 030. 지도 팝업 수정, 좌표 정밀도 개선, 역지오코딩 (2026-03-22)

## 화석산지 지도 팝업 수정
- 마커 hover 시 팝업 표시 추가 (기존: 클릭만)
- `makePopup` null/undefined 방어 처리 (`risk_score`, `site_name` 등)
- 평가 없는 산지는 우선순위 행 숨김
- GeoJSON API에서 `risk_score` null → 0 으로 변경

## FossilSite 좌표 정밀도 개선
- `import_sites_from_kprdb`에서 그룹핑 키(4자리)를 저장값으로 사용하던 문제 수정
- 원본 DMS 데이터(초 단위 소수점 2자리)에서 변환한 6자리 decimal로 저장하도록 변경
- 기존 38건 좌표 일괄 업데이트 (4자리 → 6자리)
  - 예: `(33.2439, 126.56)` → `(33.243917, 126.560028)`

## 주소 파싱 개선
- `parse_location`에 한국 행정구역 키워드 검증 추가
- 영문 설명(예: "15 m above the base of...")이 시도/시군구에 잘못 들어가던 문제 수정
- 잘못 파싱된 2건 수정 (FS-2026-0005, FS-2026-0050)

## 역지오코딩 (Nominatim/OSM)
- 주소가 비어있는 FossilSite 3건에 대해 Nominatim reverse geocoding 적용
- 위경도 → 시도/시군구/읍면동 자동 변환
- site_name이 좌표뿐인 경우 지명으로 교체

## 버전 이력
| 버전 | 내용 |
|------|------|
| 0.2.22 | 지도 팝업 hover/클릭 수정, makePopup null 방어 |
| 0.2.23 | 좌표 6자리, 주소 파싱 개선, 역지오코딩 |
