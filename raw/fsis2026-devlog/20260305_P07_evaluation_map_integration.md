# P07: 위험도 평가 페이지 지도 통합 및 자동 스크린샷

**작성일**: 2026-03-05
**목표**: gheval 데스크톱 앱의 지도/도로거리/스크린샷 기능을 웹 평가 페이지에 이식

## 현황 분석

### 현재 상태 (fsis2026)
- 평가 등록 페이지(`evaluation_form.html`)에 **지도 없음** — 숫자 입력 폼만 존재
- 도로거리 자동측정: `자동측정` 버튼 → AJAX로 Overpass API 호출 → 거리값만 반환
- 스크린샷: 사용자가 **직접 파일을 업로드**해야 함 (3개까지)
- 도로 snap 좌표(`road_snap_lat/lng`)가 API 응답에 포함되지만 **지도에 표시되지 않음**

### gheval 데스크톱 앱 기능
- Leaflet.js 지도에 산지 마커 + 도로까지 빨간 점선 + 거리 라벨 표시
- OSM / Esri 위성 / Wayback 여름 위성 / Hybrid 4가지 지도 유형
- `QWebEngineView.grab()`으로 지도 뷰포트를 PNG 캡처 → 스크린샷 자동 저장
- 분석 반경 원(circle) 표시

---

## 구현 계획

### Phase 1: 평가 폼에 Leaflet 지도 삽입

**파일**: `evaluator/templates/evaluator/evaluation_form.html`

1. 평가 폼 상단(사이트 정보 아래)에 Leaflet 지도 패널 추가
   - 높이 400px, 전체 너비
   - 산지 좌표(`site.latitude`, `site.longitude`)에 마커 표시
   - 기본 줌 레벨 15

2. 지도 레이어 전환 버튼 (4가지)
   - ROADMAP: OpenStreetMap 타일
   - SKYVIEW: Esri World Imagery
   - SKYVIEW (Summer): Esri Wayback (여름 촬영분)
   - HYBRID: Esri 위성 + OSM 라벨 오버레이

3. 위성 이미지 촬영일 표시
   - 좌하단 정보 박스에 Esri 메타데이터 쿼리 → 촬영일 표시
   - gheval의 `fetchCurrentDate()` / `fetchWaybackDate()` 로직 그대로 이식

**참조**: `gheval/templates/map_view.html` (타일 레이어, 지도 타입 전환)

---

### Phase 2: 도로거리 측정 + 지도 시각화

**파일**: `evaluation_form.html` (JS), `evaluator/views.py`

1. `자동측정` 버튼 클릭 시:
   - 기존 AJAX API 호출 (이미 구현됨)
   - 응답의 `snap_lat`, `snap_lng`을 이용하여 **지도에 빨간 점선** 표시
   - 산지 마커 → 도로 snap 포인트까지 `L.polyline` (빨강, dashArray)
   - 도로 snap 포인트에 `L.circleMarker` (빨강)
   - 중간 지점에 거리 라벨 (`L.divIcon`)
   - gheval의 `showRoadLine()` 함수 그대로 이식

2. `road_snap_lat`, `road_snap_lng` 값을 hidden input에 저장
   - 폼 제출 시 DB에 함께 저장되도록 `SiteEvaluationForm`에 필드 추가

**참조**: `gheval/templates/map_view.html:332-358` (showRoadLine 함수)

---

### Phase 3: 지도 스크린샷 자동 캡처

**핵심 기술**: `leaflet-image` 라이브러리 또는 `html2canvas`

데스크톱(gheval)은 `QWebEngineView.grab()`을 사용하지만, 웹에서는 클라이언트 사이드 캡처가 필요.

#### 방법 A: `leaflet-image` (권장)
- Leaflet 지도를 canvas로 렌더링하여 PNG blob 생성
- CDN: `https://unpkg.com/leaflet-image@0.4.0/leaflet-image.js`
- **장점**: Leaflet 전용, 타일/마커/폴리라인 포함 캡처
- **제한**: 일부 CORS 이슈 가능 (타일 서버 의존)

#### 방법 B: `html2canvas` (fallback)
- DOM 요소를 canvas로 렌더링
- 타일 이미지 CORS 처리 필요 (`useCORS: true`)

#### 구현 방식

1. **자동 캡처 타이밍**: 도로거리 측정 완료 후 자동으로 스크린샷 생성
   - 도로 점선이 표시된 상태의 지도 캡처
   - 각 지도 유형(HYBRID, ROADMAP)별로 자동 캡처 가능

2. **캡처 → 서버 업로드 흐름**:
   ```
   leafletImage(map, callback) → canvas.toBlob() → FormData → fetch POST
   ```

3. **새 API 엔드포인트**: `POST /evaluator/api/upload-screenshot/<evaluation_pk>/`
   - Base64 또는 multipart로 이미지 수신
   - `EvaluationScreenshot` 레코드 자동 생성
   - map_type, zoom_level 메타데이터 함께 저장

4. **평가 등록 폼 흐름 변경**:
   - 기존: 사용자가 직접 파일 업로드 (3개 formset)
   - 변경: 도로거리 측정 시 자동 캡처 + 수동 캡처 버튼도 제공
   - 캡처된 이미지를 미리보기로 표시 (폼 하단)
   - 저장 시 hidden input의 blob/base64를 함께 제출

#### 대안: 서버 사이드 캡처 (간단한 접근)
- 클라이언트 캡처가 CORS로 어려울 경우
- 서버에서 `staticmap` API 또는 `selenium`으로 캡처
- 단, 도로 점선 등 동적 요소는 포함 불가

**권장 접근**: 클라이언트 `leaflet-image` 우선 시도, CORS 문제 시 `html2canvas` fallback

---

### Phase 4: 분석 반경 원(circle) 표시

**파일**: `evaluation_form.html`

1. 토지피복 분석 반경 입력값(250/500/1000m) 변경 시
   - 지도에 `L.circle` 표시 (하늘색 점선, 반투명)
   - gheval의 `showAnalysisCircle()` 함수 이식

2. 반경 원 내부의 위성 이미지를 볼 수 있도록 지도 줌 자동 조정

**참조**: `gheval/templates/map_view.html:366-381`

---

## 파일 변경 목록

| 파일 | 변경 내용 |
|------|----------|
| `evaluator/templates/evaluator/evaluation_form.html` | Leaflet 지도 추가, 도로 점선 표시, 스크린샷 캡처 JS |
| `evaluator/views.py` | 스크린샷 업로드 API 엔드포인트 추가 |
| `evaluator/urls.py` | 스크린샷 업로드 URL 추가 |
| `evaluator/forms.py` | `road_snap_lat/lng` hidden 필드 추가 |

## 추가 의존성

- `leaflet-image` (CDN, JS만) — 또는 `html2canvas` (CDN)
- Leaflet.js, Esri 타일 — 이미 fossil_site_map.html에서 사용 중

## 구현 순서

1. **Phase 1** (지도 삽입) — 독립적, 바로 시작 가능
2. **Phase 2** (도로 시각화) — Phase 1 완료 후
3. **Phase 3** (스크린샷 자동화) — Phase 2 완료 후
4. **Phase 4** (분석 원) — Phase 1 완료 후 (Phase 2와 병행 가능)

## 예상 결과

평가 등록 페이지에서:
- 산지 위치를 Leaflet 지도로 확인
- `자동측정` 클릭 → 도로까지 빨간 점선 + 거리 라벨이 지도에 표시
- 도로 측정 완료 후 현재 지도 상태가 자동으로 스크린샷 캡처
- 수동 `캡처` 버튼으로 추가 스크린샷도 가능
- 캡처된 이미지가 미리보기로 표시되고, 저장 시 자동 첨부
- 기존 파일 업로드 formset은 유지 (직접 촬영 사진 등)
