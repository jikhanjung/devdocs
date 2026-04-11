# 지도 시스템

화석산지와 표본 위치를 시각화하는 지도 인프라. Kakao Maps에서 시작하여 MapLibre GL JS로 통합되었다.

## 지도 전환 이력

```
Kakao Maps API → Leaflet.js 1.9.4 → MapLibre GL JS 4.7.1
```

### Kakao Maps (초기)
- fossil_map: 92개 표본 위치 GeoJSON API
- 550MB map_tiles 정적 파일

### Leaflet.js
- evaluator 앱의 fossil_site_map에 도입
- MarkerCluster 클러스터링
- leaflet-image / html2canvas 스크린샷 캡처
- 4가지 지도 타입 (ROADMAP/SKYVIEW/HYBRID/Esri Wayback)

### MapLibre GL JS (현재)
- WebGL 벡터 렌더링
- GeoJSON source + cluster: true
- style JSON 기반 설정
- [lng, lat] 좌표 순서 (Leaflet의 [lat, lng]과 반대)
- 줌 레벨 매핑: kakao_level ≈ 21 - maplibre_zoom
- glyphs 설정 (텍스트 렌더링)
- preserveDrawingBuffer (캔버스 캡처)
- raster tile source 지원

## 주요 지도 페이지

| 페이지 | 설명 |
|---|---|
| fossil_map | GeoJSON 클러스터 기반 표본 지도 |
| fossil_site_map | P0-P4 등급별 마커 색상, "미평가" 필터 |
| site_detail | 개별 산지 지도 |
| evaluation_form | 평가 폼 내장 지도 (가장 복잡) |
| /public/map | 공개 플랫폼 지도 |
| site_candidate_map | 산지 후보 지도 |

## 지도 UI 개선사항

- 팝업: hover + click 지원, sticky vs non-sticky 관리
- 마커: circle-radius/stroke 스타일링, P0-P4 색상
- 좌표 정밀도: 4자리 → 6자리 소수점
- 주소 파싱: 한국 행정구역 검증
- 역지오코딩: Nominatim/OSM (누락 주소 3건 보완)

## 관련 페이지

- [평가 시스템](evaluator.md) — 지도 기반 화석산지 평가
- [프로젝트 개요](fsis2026-overview.md)

---
*Sources: 002, 008, 012, 030, 033, P07, P08*
