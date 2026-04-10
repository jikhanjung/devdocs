# Evaluator (화석산지 평가 시스템)

화석산지의 보존 위험도를 정량적으로 평가하는 시스템. 국제 표준인 Brilha(2016) 모델을 기반으로 한다.

## Brilha 5항목 위험도 평가

| 항목 | 가중치 |
|---|---|
| fragility (취약성) | 35% |
| proximity_hazard (위험 근접도) | 20% |
| legal_protection (법적 보호) | 20% |
| accessibility (접근성) | 15% |
| population_density (인구밀도) | 10% |

**risk_score** = 가중합 → P0~P4 우선순위 등급 산출

## 주요 모델

| 모델 | 역할 |
|---|---|
| **FossilSite** | 화석산지 (자동 생성 site_code: FS-YYYY-NNNN) |
| **SpecimenAssessment** | 표본별 평가 |
| **EvaluationImage** | 평가 이미지 (screenshot/field photo/other 분류) |

## 지도 기능

- **Leaflet.js 1.9.4** + MarkerCluster (이후 MapLibre GL JS로 전환)
- GeoJSON API 기반 클러스터링
- 4가지 지도 유형 (ROADMAP/SKYVIEW/HYBRID/Esri Wayback)
- P0~P4 등급별 마커 색상, "미평가" 필터
- fossil_site_map: 전체 화석산지 지도
- 공개 플랫폼: /public/ (민감 데이터 마스킹)

## gheval 포팅 기능

기존 gheval 데스크톱 앱에서 포팅한 기능:

- **좌표 파서**: DMS, DDM, 한국식, 십진수 형식 지원
- **도로 거리 계산**: Overpass API 기반, 빨간 폴리라인 + 스냅 포인트 시각화
- **토지피복 분석**: PIL/NumPy 기반 HSV+ExG 알고리즘 (OpenCV 미사용)
  - 5개 분류: dense vegetation, sparse vegetation, bare, built, water
  - 서버사이드 위성 이미지 분석 API 엔드포인트

## 평가 폼 UX

- 스크린샷 폼: 동적 행 추가/삭제, 이미지 lightbox 팝업
- Brilha 항목별 방향성 설명 표시
- 평가일 자동 설정 (오늘 날짜)
- 산지 좌표 표시

## 관련 페이지

- [지도 시스템](map-system.md) — 지도 인프라 상세
- [프로젝트 개요](overview.md)

---
*Sources: 008, 009, P06, P07, P08, P09*
