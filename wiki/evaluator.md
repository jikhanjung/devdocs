# GeoHeritage Evaluator (화석산지 평가 시스템)

[KOFHIN](fsis2026-overview.md) 프로젝트의 **3번째 핵심 목표**. 화석산지의 보존 위험도를 정량적으로 평가하고, 훼손 가능성·학술 가치·실행 가능성을 결합하여 발굴 우선순위(P0~P4)를 도출하는 GIS 통합 플랫폼. 공식 제안서상 이름은 **GeoHeritage Evaluator(가칭)**이며, 구현 앱명은 `evaluator`.

국제 표준인 **Brilha (2016)** 모델을 기반으로 하며, P0~P4 매트릭스는 **Sevieri et al. (2020)**의 다중재해 위험 우선순위화 프레임워크를 따른다.

## Brilha (2016) 5항목 정량 위험도 평가

각 항목 1~4점, 가중합으로 Risk Score 산정. **점수가 높을수록 위험이 크다.**

| 평가 항목 | 가중치 | 점수 기준 |
|---|---|---|
| **지질요소 훼손(열화) 가능성** (fragility) | **35%** | 4점: 전체 훼손 가능성 높음<br>3점: 주요 요소 훼손 가능<br>2점: 부차적 요소 훼손 가능<br>1점: 부차적 요소도 훼손 가능성 낮음 |
| **훼손 유발 활동·지역과의 근접성** (proximity_hazard) | **20%** | 4점: 50m 미만<br>3점: 200m 미만<br>2점: 500m 미만<br>1점: 1km 미만 |
| **법적 보호 및 접근 통제** (legal_protection) | **20%** | 4점: 보호 없음 + 접근통제 없음<br>3점: 보호 없음 + 접근통제 있음<br>2점: 보호 있음 + 접근통제 없음<br>1점: 보호 있음 + 접근통제 있음 |
| **접근성** (accessibility) | **15%** | 4점: 포장도로 100m 미만 + 버스 주차 가능<br>3점: 포장도로 500m 미만<br>2점: 비포장도로로 버스 접근 가능<br>1점: 버스 가능 도로에서 1km 미만 |
| **인구 밀도** (population_density) | **10%** | 4점: 1,000명/km² 초과<br>3점: 250~1,000명/km²<br>2점: 100~250명/km²<br>1점: 100명/km² 미만 |

> **현재 구현 상태** (devlog 008/009, 2026-03-05~06): `SiteEvaluation` 모델과 평가 폼 모두 **Brilha 5개 항목을 전부** 지원한다. 사용자가 폼에서 5개 점수를 입력하면 가중합으로 `risk_score`가 자동 계산되고 P0~P4 우선순위가 자동 배정된다.
>
> **자동 측정(auto-measurement)**: `accessibility`(Overpass API 도로거리)만 자동화되어 있고, 보조적으로 토지피복 분석(PIL+numpy HSV+ExG, 5개 분류)이 사용된다. 나머지 4개 항목(`fragility`, `proximity_hazard`, `legal_protection`, `population_density`)은 평가 관리자가 폼에서 수동 점수를 입력한다. 이는 Brilha 기준의 본성상 자연스러운 접근이다 — 훼손 가능성·법적 보호·인구 밀도는 문맥 판단이 필요하므로 자동화 대상이 아니다.
>
> **실제 평가 단계는 아직 시작되지 않았다.** KOFHIN 프로젝트는 단계적으로 진행되며, 2026-04 현재는 **PDF 확보 단계**이고 다음은 **정보 입력 단계**(화석산지·taxa·specimens — UI는 이미 준비됨)이다. **화석산지 평가 단계**는 몇 달 뒤에 본격적으로 시작될 예정이다. 즉 `evaluator` 앱은 **평가 단계에 앞서 미리 준비되어 있는 상태**이며, 현재 미사용 상태는 개발 지연이 아니라 phase 일정에 맞춰 대기 중이라는 뜻이다.
>
> 참고: `docs/PROJECT_PROPOSAL.md`(2026-02-22)에는 "gheval 현재 구현은 접근성과 식생피복만 반영"이라는 각주가 있는데, 이 각주는 **원본 gheval PyQt6 데스크톱 앱**(P06-3에서 Django로 포팅된 조상 프로젝트)에 대한 것이다. KOFHIN의 `evaluator` 앱은 그 뒤(2026-03-05, P06-2)에 새로 만들어지면서 5개 항목을 모두 포함한 상태로 출발했다.

**risk_score** = 가중합 → P0~P4 우선순위 등급 산출

## 발굴 우선순위 (P0~P4)

우선순위는 단일 기준이 아닌 **Risk × Value × Feasibility** 3축 결합으로 결정 (Sevieri et al., 2020).

| 축 | 출처 | 설명 |
|---|---|---|
| **Risk** (훼손·멸실 위험도) | GeoHeritage Evaluator 산출값 | Brilha 2016 5항목 가중합 |
| **Value** (학술·문화 가치) | 표본 DB 중요성 평가 | 높음/중간/낮음 3등급 |
| **Feasibility** (실행가능성) | 현장 평가 | 인허가·접근·안전·보존처리 여건 |

### 우선순위 매트릭스 (Risk × Value)

|  | 가치 낮음 | 가치 중간 | 가치 높음 |
|---|---|---|---|
| **위험도 높음** | **P2** 방어/기록 우선, 긴급 훼손저감, 발굴 제한적 | **P1** 긴급조치+선별발굴, 1~2년 내 착수 | **P0** 최우선 긴급발굴/보존, 즉시 착수 |
| **위험도 중간** | **P3** 모니터링 중심 | **P2** 단계적 조사, 정밀조사 후 발굴 검토 | **P1** 계획발굴 우선, 2~3년 내 배치 |
| **위험도 낮음** | **P4** 관찰·기록, 예산 최소 | **P3** 기초연구/활용연계 | **P2** 장기 보전·활용, 관광/교육 적합 |

### 우선순위 정의

| 등급 | 정의 | 조치 |
|---|---|---|
| **P0** | 최우선·긴급 대응형 | 즉시 정밀조사·긴급발굴(또는 긴급 보존조치) 착수 |
| **P1** | 우선 추진형 | 단기(1~2년) 내 발굴·연구 우선 배치, 보존·관리 계획 병행 |
| **P2** | 단계 추진형 | 중기(2~3년) 내 정밀조사 후 필요 시 발굴 전환 |
| **P3** | 모니터링·기초조사형 | 정기 모니터링·기초조사 중심, 상황 변화 시 상향 조정 |
| **P4** | 관찰·기록 유지형 | DB 기록 유지와 최소 관찰, 예산 최소 투입 |

## 주요 모델

| 모델 | 역할 |
|---|---|
| **FossilSite** | 화석산지 (자동 생성 site_code: FS-YYYY-NNNN), 향후 **GeoSite-ID**와 연동 예정 |
| **SpecimenAssessment** | 표본별 평가 |
| **EvaluationImage** | 평가 이미지 (screenshot/field photo/other 분류) |

## GeoHeritage Evaluator 주요 기능 (제안서)

- 화석산지 DB 지도 레이어 조회·검색·필터링
- **위성영상 시계열 비교** (ESRI Wayback 활용): 공사 흔적·범람원 확대 등 지표 변화 기록
- 토지피복 변화 및 개발 흔적 모니터링
- 모니터링 결과 → Risk Score 자동 입력 연계
- 발굴 우선순위(P0~P4) 시각화 및 대시보드
- 리포트 출력 기능
- **대상**: DB 구축 화석산지 중 최대 200곳

## 지도 기능

- **Leaflet.js 1.9.4** + MarkerCluster (이후 MapLibre GL JS로 전환)
- GeoJSON API 기반 클러스터링
- 4가지 지도 유형 (ROADMAP / SKYVIEW / HYBRID / **Esri Wayback**)
- P0~P4 등급별 마커 색상, "미평가" 필터
- fossil_site_map: 전체 화석산지 지도
- 공개 플랫폼: `/public/` (민감 좌표 마스킹 — **좌표 공개 수준 차등화**)

### 좌표 공개 수준 (제안서 정책)

| 수준 | 의미 |
|---|---|
| A | 노두/지점 수준 (공개) |
| B | 리/면 단위 (부분 공개) |
| C | 비공개 대체좌표 (훼손 위험 방지, 불법채집 차단) |

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
- **학명 이탤릭 렌더링** (068): `scientific_name` 필드의 `<i>...</i>` 태그를 `|safe` 필터로 이탤릭 렌더링 (type_specimen_list.html, specimen_assessment_detail.html, specimen_assessment_form.html, site_specimen_list.html 적용)

## 참고 문헌

- **Brilha, J. (2016)**. *Inventory and quantitative assessment of geosites and geodiversity sites: a review*. Geoheritage 8(2): 119–134. → 5항목 위험도 평가 출처
- **Sevieri, G. et al. (2020)**. *A multi-hazard risk prioritisation framework for cultural heritage assets*. NHESS 20(5): 1391–1414. → P0~P4 매트릭스 출처
- UNESCO (2015). *Operational Guidelines for UNESCO Global Geoparks*.
- UNESCO World Heritage Centre (2025). *Operational Guidelines for the Implementation of the World Heritage Convention*.

## 관련 페이지

- [프로젝트 개요](fsis2026-overview.md) — 사업 전체 맥락
- [지도 시스템](map-system.md) — 지도 인프라 상세
- [KOFHIN 매뉴얼](kofhin-manuals.md) — 위험도 평가 운영 가이드
- [KPRDB](kprdb.md) — Reference-ID 연결
- [GHDB](ghdb.md) — Specimen-ID 연결

---
*Sources: **008 (SiteEvaluation 5 factors + risk_score 자동 계산 + P0~P4 자동 배정, 2026-03-05)**, **009 (Brilha 5개 항목 폼 UI + 점수 방향 설명, 2026-03-06)**, P06, P07, P08, P09, 068 (italic), docs/PROJECT_PROPOSAL.md (Brilha 정확 가중치 + P0~P4 매트릭스 + 좌표 공개 수준; gheval 각주는 원본 PyQt6 앱 대상), docs/admin_manual.md (위험도 평가 운영 섹션).*
