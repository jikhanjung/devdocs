# P08: 모든 지도를 MapLibre GL JS로 통합

**작성일**: 2026-03-05
**상태**: 완료 (2026-03-05)
**목표**: Leaflet.js + 카카오맵을 모두 MapLibre GL JS로 교체하여 단일 지도 라이브러리로 통합

## 왜 MapLibre인가

- WebGL 기반 벡터 렌더링 → 대량 마커/타일 처리 시 훨씬 빠름
- 벡터 타일 네이티브 지원 (향후 확장성)
- GeoJSON 소스 기반 내장 클러스터링 → MarkerCluster 플러그인 불필요
- 오픈소스 (BSD), Mapbox GL JS 포크, 활발한 커뮤니티
- 카카오맵 API 키 의존성 제거 (외부 서비스 의존도 감소)

## 대상 파일 (7개)

### Leaflet 사용 (4개)

| # | 파일 | 복잡도 | 핵심 기능 |
|---|------|--------|----------|
| 1 | `fsis/templates/fsis/public/site_detail.html` | 낮음 | 단일 마커, 팝업, 300px 미니맵 |
| 2 | `evaluator/templates/evaluator/evaluation_form.html` | 높음 | 4종 타일, 도로 점선, 분석 원, Wayback, 스크린샷 캡처 |
| 3 | `fsis/templates/fsis/fossil_site_map.html` | 높음 | MarkerCluster, 4종 필터, 사이드바, P0-P4 마커 |
| 4 | `fsis/templates/fsis/public/map.html` | 높음 | MarkerCluster, 3종 필터, 사이드바, 보호상태 마커 |

### 카카오맵 사용 (3개)

| # | 파일 | 복잡도 | 핵심 기능 |
|---|------|--------|----------|
| 5 | `fsis/templates/fsis/fossil_map.html` | 높음 | 카카오맵 + 지질도 타일 오버레이 + 표본 마커/InfoWindow + 클러스터 + 동적 JSON 로드 |
| 6 | `fsis/static/fsis/nkfadmin_kakao.html` | 높음 | 구 정적 페이지 (fossil_map.html의 원본, 참조용) |
| 7 | `fsis/static/fsis/nkfadmin_occurrence.js` | 중간 | `nk_fossil_location` 클래스 (카카오맵 Marker/InfoWindow/CustomOverlay) |

### 카카오맵 링크만 사용 (1개)

| # | 파일 | 변경 |
|---|------|------|
| 8 | `fsis/templates/fsis/museum_specimen_detail.html` | `kakao_map_url` 링크 → 자체 지도로 대체 또는 제거 |
| 9 | `fsis/models.py` | `kakao_map_url` 프로퍼티 제거 |

---

## API 매핑

### Leaflet → MapLibre

| Leaflet | MapLibre GL JS |
|---------|----------------|
| `L.map('id', {center, zoom, layers})` | `new maplibregl.Map({container, center, zoom, style})` |
| `L.tileLayer(url, opts)` | 스타일 내 `sources` + `layers` 정의 |
| `L.marker([lat, lng])` | `new maplibregl.Marker().setLngLat([lng, lat])` |
| `L.popup()` | `new maplibregl.Popup()` |
| `L.polyline(coords, style)` | GeoJSON source + `line` layer |
| `L.circle([lat,lng], {radius})` | GeoJSON polygon (64각형 근사) → `fill`/`line` layer |
| `L.circleMarker()` | GeoJSON source + `circle` layer |
| `L.divIcon({html})` | `maplibregl.Marker({element: dom})` |
| `L.control.scale()` | `new maplibregl.ScaleControl()` |
| `MarkerCluster` | GeoJSON source `cluster: true` (내장) |
| `map.fitBounds(bounds)` | `map.fitBounds([[w,s],[e,n]])` |
| `map.setView([lat,lng], zoom)` | `map.flyTo({center:[lng,lat], zoom})` |

### 카카오맵 → MapLibre

| 카카오맵 | MapLibre GL JS |
|---------|----------------|
| `new kakao.maps.Map(container, {center, level})` | `new maplibregl.Map({container, center, zoom, style})` |
| `kakao.maps.LatLng(lat, lng)` | `[lng, lat]` (배열) |
| `new kakao.maps.Marker({position})` | `new maplibregl.Marker().setLngLat([lng, lat])` |
| `new kakao.maps.InfoWindow({content})` | `new maplibregl.Popup().setHTML(content)` |
| `new kakao.maps.CustomOverlay({position, content})` | `new maplibregl.Marker({element: dom})` |
| `kakao.maps.LatLngBounds()` | `new maplibregl.LngLatBounds()` |
| `map.setBounds(bounds)` | `map.fitBounds(bounds)` |
| `kakao.maps.event.addListener(obj, 'click', fn)` | `marker.getElement().addEventListener('click', fn)` |
| `kakao.maps.Tileset` (커스텀 타일 오버레이) | `map.addSource('tiles', {type:'raster', tiles:[url]})` + layer |
| `map.addOverlayMapTypeId()` | layer visibility 토글 |
| `map.getLevel()` / `setMinLevel` / `setMaxLevel` | `map.getZoom()` / `minZoom` / `maxZoom` 옵션 |
| `kakao.maps.ZoomControl()` | `new maplibregl.NavigationControl()` |

**좌표 순서 주의**: Leaflet/카카오맵은 `[lat, lng]` / `LatLng(lat, lng)`, MapLibre는 `[lng, lat]`

**줌 레벨 매핑**: 카카오맵 level과 MapLibre zoom은 반비례
- 카카오 level 13 (전국) ≈ MapLibre zoom 7
- 카카오 level 9 ≈ MapLibre zoom 10
- 카카오 level 1 (최대확대) ≈ MapLibre zoom 19
- 변환: `maplibre_zoom ≈ 21 - kakao_level` (근사)

## CDN

```html
<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css" />
<script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
```

---

## 구현 계획

### Phase 1: site_detail.html (가장 간단, 패턴 확립)

가장 단순한 파일부터 시작하여 MapLibre 사용 패턴을 확립.

**변환 내용:**
- `L.map` → `maplibregl.Map` (래스터 타일 스타일)
- `L.marker` → `maplibregl.Marker`
- `L.popup` → `maplibregl.Popup`
- Leaflet CSS/JS CDN → MapLibre CDN

**래스터 타일 스타일 정의:**
```javascript
style: {
    version: 8,
    sources: {
        osm: {
            type: 'raster',
            tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
            tileSize: 256,
            attribution: '&copy; OSM'
        }
    },
    layers: [{ id: 'osm', type: 'raster', source: 'osm' }]
}
```

---

### Phase 2: evaluation_form.html (가장 복잡)

**변환 항목:**

1. **지도 초기화**: 래스터 타일 스타일 + `preserveDrawingBuffer: true`
2. **타일 레이어 전환 (4종)**:
   - 모든 래스터 소스를 초기에 등록하고 layer visibility를 토글
   - `setStyle()` 사용 시 오버레이 재추가 필요하므로 visibility 토글 권장
3. **도로 점선**: GeoJSON LineString source + `line` layer (`line-dasharray: [6, 4]`)
4. **도로 끝점 마커**: GeoJSON Point source + `circle` layer
5. **거리 라벨**: `maplibregl.Marker({element: div})` — DOM 마커
6. **분석 반경 원**: 64각형 폴리곤 GeoJSON → `fill` + `line` layer
7. **Imagery info overlay**: 커스텀 컨트롤 `map.addControl()`
8. **Wayback 레이어**: 동적 래스터 소스 추가
9. **스크린샷 캡처**: `map.getCanvas().toDataURL()` 네이티브! → **html2canvas 제거**

---

### Phase 3: fossil_site_map.html (관리자 지도)

**변환 항목:**

1. **MarkerCluster → 내장 클러스터링**:
   ```javascript
   map.addSource('sites', {
       type: 'geojson', data: geojsonData,
       cluster: true, clusterMaxZoom: 14, clusterRadius: 50
   });
   ```
2. **P0-P4 색상 마커**: `circle` 레이어 + `match` expression
3. **팝업**: `map.on('click', 'unclustered', ...)`
4. **필터링**: `map.setFilter()` — 마커 재생성 없이 즉시 필터
5. **사이드바**: 현재 로직 유지, `map.flyTo()` + 팝업
6. **타일 레이어 전환**: visibility 토글

---

### Phase 4: public/map.html (공개 지도)

Phase 3과 동일 패턴. 차이점:
- 보호상태별 색상 (priority 대신)
- 필터 3종 (priority 필터 없음)

---

### Phase 5: fossil_map.html + nkfadmin_occurrence.js (카카오맵 → MapLibre)

**가장 큰 변경**. 카카오맵 전용 코드를 완전 교체.

**현재 구조:**
- `fossil_map.html`: 카카오맵 초기화 + 지질도 커스텀 타일 오버레이 + JSON API 로드
- `nkfadmin_occurrence.js`: `nk_fossil_location` 클래스 — 카카오맵 Marker/InfoWindow/CustomOverlay 래퍼
- 지질도 타일: `fsis/static/fsis/map_tiles/{z}/{x}_{y}.png` (550MB, 카카오맵 타일셋 형식)

**변환 내용:**

1. **카카오맵 → MapLibre Map 초기화**
   - `kakao.maps.Map` → `maplibregl.Map`
   - 카카오 level 13 → MapLibre zoom 7
   - `kakao.maps.ZoomControl` → `maplibregl.NavigationControl`

2. **지질도 타일 오버레이** (핵심 과제):
   - 카카오맵 `Tileset`은 카카오 좌표계 기반 타일 인덱스 사용
   - MapLibre는 표준 TMS/XYZ 타일 인덱스 사용
   - **타일 좌표 변환이 필요하거나, 타일을 재생성해야 할 수 있음**
   - 카카오맵 타일셋: `getTile(x, y, z)` → `{z}/{x}_{y}.png`
   - 확인 필요: 카카오맵의 x,y,z와 표준 XYZ/{z}/{x}/{y}가 같은 좌표계인지
   - **옵션 A**: 타일이 표준 XYZ와 호환되면 → `map.addSource('geolmap', {type:'raster', tiles:['.../{z}/{x}_{y}.png']})` 로 직접 사용
   - **옵션 B**: 좌표계가 다르면 → 타일 재생성 스크립트 작성 또는 서버 사이드 프록시

3. **nk_fossil_location 클래스 재작성**:
   - `kakao.maps.Marker` → `maplibregl.Marker`
   - `kakao.maps.InfoWindow` → `maplibregl.Popup`
   - `kakao.maps.CustomOverlay` → `maplibregl.Marker({element})` 또는 제거
   - `kakao.maps.LatLng` → `[lng, lat]` 배열
   - `kakao.maps.LatLngBounds` → `maplibregl.LngLatBounds`
   - `kakao.maps.event.addListener` → DOM event listeners

4. **JSON 데이터 로드**: `fetch('{% url "fsis:fossil_map_data" %}')` — 그대로 유지

5. **UI 옵션 (Fit View, Cluster, Auto Close, GeolMap)**: 로직 유지, API만 교체

---

### Phase 6: 카카오맵 링크 제거

1. `museum_specimen_detail.html`: `[카카오맵]` 링크 → 제거 또는 자체 지도 페이지 링크로 대체
2. `fsis/models.py`: `kakao_map_url` 프로퍼티 제거
3. `nkfadmin_kakao.html`: 구 참조 파일 — 삭제 또는 보관

---

## 지질도 타일 좌표 호환성 확인 (Phase 5 사전 조사)

카카오맵의 `Tileset.getTile(x, y, z)` 좌표계를 확인해야 함:

```javascript
// 현재 코드
kakao.maps.Tileset.add('TILE_NUMBER',
    new kakao.maps.Tileset({
        width: 256, height: 256,
        getTile: function(x, y, z) {
            // z: 7~13, 카카오맵 내부 좌표
            img.src = MAP_TILES_BASE + '/' + z + '/' + x + '_' + y + '.png';
        }
    }));
map.setMinLevel(7);  // 카카오 level 7 = 가장 확대된 상태
map.setMaxLevel(13); // 카카오 level 13 = 전국
```

- 카카오맵 level은 MapLibre zoom과 **반비례** (level 높을수록 축소)
- 타일 z 값이 카카오 level과 같은지, 또는 별도 좌표계인지 확인 필요
- **확인 방법**: 실제 타일 파일 구조를 보고 z=7~13 폴더의 x,y 범위를 분석

---

## 공통 스타일 유틸리티

```javascript
// static/js/map_styles.js
const MAP_STYLES = {
    ROADMAP: {
        version: 8,
        sources: { osm: { type: 'raster', tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'], tileSize: 256, attribution: '&copy; OSM' }},
        layers: [{ id: 'osm', type: 'raster', source: 'osm' }]
    },
    SKYVIEW: {
        version: 8,
        sources: { esri: { type: 'raster', tiles: ['https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}'], tileSize: 256, attribution: '&copy; Esri' }},
        layers: [{ id: 'satellite', type: 'raster', source: 'esri' }]
    },
};
```

---

## 구현 순서

```
Phase 1 (site_detail)     ← 간단, 패턴 확립
  ↓
Phase 2 (evaluation_form) ← 복잡/핵심
  ↓
Phase 3 (fossil_site_map) ← 클러스터링
  ↓
Phase 4 (public/map)      ← Phase 3 복사
  ↓
Phase 5 (fossil_map + occurrence.js) ← 카카오맵 전환 (지질도 타일 오버레이는 보류)
  ↓
Phase 6 (카카오맵 링크/코드 정리)
```

**참고**: 지질도 맵타일은 카카오맵 좌표계 기반이라 호환 안 될 가능성 높음.
타일 재생성은 별도 작업으로 보류하고, Phase 5에서는 지질도 오버레이 없이 전환.

## 제거되는 외부 의존성

| 제거 대상 | 파일 수 |
|-----------|---------|
| `leaflet@1.9.4` CSS/JS | 2 |
| `leaflet.markercluster@1.5.3` CSS 2 + JS 1 | 3 |
| `html2canvas@1.4.1` JS | 1 |
| 카카오맵 SDK (`dapi.kakao.com`) | 1 |
| `nkfadmin_occurrence.js` (카카오맵 전용) | 1 |
| **합계** | **8개 → 2개 (maplibre-gl CSS/JS)** |

## 리스크

1. **지질도 타일 좌표**: 카카오맵 전용 좌표계일 경우 타일 재생성 필요 (수시간 소요)
2. **nkfadmin_kakao.html**: 구 정적 파일, 참조용으로 보존

---

## 작업 결과 (2026-03-05)

### 변경된 파일 (8개)

| Phase | 파일 | 변경 내용 |
|-------|------|----------|
| 1 | `fsis/templates/fsis/public/site_detail.html` | Leaflet → MapLibre (마커, 팝업, ScaleControl) |
| 2 | `evaluator/templates/evaluator/evaluation_form.html` | Leaflet → MapLibre (4종 타일 visibility 토글, 도로선 GeoJSON layer, 분석원 64각형 polygon, Wayback 동적 source, **`map.getCanvas().toDataURL()` 네이티브 캡처**) |
| 3 | `fsis/templates/fsis/fossil_site_map.html` | Leaflet+MarkerCluster → MapLibre 내장 `cluster:true` + `match` expression으로 P0-P4 색상 |
| 4 | `fsis/templates/fsis/public/map.html` | 동일 패턴 (보호상태별 색상) |
| 5 | `fsis/templates/fsis/fossil_map.html` | **카카오맵 → MapLibre** (지질도 타일 오버레이 보류) |
| 5 | `fsis/static/fsis/nkfadmin_occurrence.js` | `nk_fossil_location` 클래스: 카카오맵 Marker/InfoWindow/CustomOverlay → MapLibre Marker/Popup |
| 6 | `fsis/templates/fsis/museum_specimen_detail.html` | `[카카오맵]` 링크 제거 |
| 6 | `fsis/models.py` | `kakao_map_url` 프로퍼티 제거 |

### 제거된 외부 의존성 (최종)

- `leaflet@1.9.4` CSS/JS → 제거
- `leaflet.markercluster@1.5.3` CSS 2개 + JS 1개 → 제거 (내장 클러스터링으로 대체)
- `html2canvas@1.4.1` JS → 제거 (`preserveDrawingBuffer: true` + `canvas.toDataURL()`)
- `카카오맵 SDK` (`dapi.kakao.com/v2/maps/sdk.js`) → 제거
- **7개 CDN/SDK → 2개** (`maplibre-gl@4.7.1` CSS/JS)

### 보류 사항

- 지질도 맵타일 (`fsis/static/fsis/map_tiles/`, 550MB): 카카오맵 좌표계 기반이라 MapLibre XYZ와 비호환 예상. 별도 타일 재생성 작업 필요.
- `nkfadmin_kakao.html`: 구 참조 파일, 삭제하지 않고 보존
