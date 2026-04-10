# 011: 지도 UI 개선

## 작업 내용

### 1. nkfadmin 파일명 리네임

예전 프로젝트 prefix(`nkfadmin`) 제거:
- `nkfadmin_occurrence.js` → `fsis_occurrence.js`
- `nkfadmin_data.js` → `fsis_data.js`
- `nkfadmin_kakao.html` → `fsis_kakao.html`
- `nkfadmin_extras.py` → 삭제 (이미 `fsis_extras.py`로 복사되어 있었음)
- 클래스명 `nk_fossil_location` → `fsis_fossil_location`
- 모든 참조 업데이트

### 2. 표본지도 (fossil_map) 마커 클러스터링

- 개별 Marker 객체 방식 → GeoJSON 소스 기반 클러스터링으로 전환
- 표본을 좌표별로 그룹핑하여 1 location = 1 feature (클러스터 숫자 = 장소 수)
- 클러스터 클릭 시 확대, 개별 포인트 클릭 시 해당 장소의 표본 목록 팝업
- 빨간색 계열 클러스터/포인트 마커, 흰색 테두리

### 3. 표본지도 navbar 추가

- 독립 HTML → `fsisbase.html` 상속으로 변경
- 다른 페이지와 동일한 메뉴 표시

### 4. 화석산지 지도 (fossil_site_map) 개선

- 개별 포인트 `circle-radius` 7 → 10, `circle-stroke-width` 2 → 2.5
- MapLibre 스타일에 `glyphs` 설정 추가 (클러스터 숫자 표시 에러 수정)
- 우선순위 필터에 "미평가" 버튼 추가 (회색 마커 필터링 가능)

### 5. 팝업 동작 수정 (fsis_occurrence.js)

- `setPopup` 제거: MapLibre 내장 클릭 토글과 수동 관리 충돌 해결
- 팝업 위치를 `setLngLat`으로 수동 지정
- 새 팝업 열 때 기존 non-sticky 팝업 자동 닫기
- 이벤트 리스너 중복 등록 방지

### 6. 메뉴 이름 변경

- "지리정보" → "표본지도" (fsisnavbar.html, base.html, index.html)
