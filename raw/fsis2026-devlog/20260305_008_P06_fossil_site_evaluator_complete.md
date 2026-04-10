# P06: 화석산지 평가 시스템 구현 완료

**작성일**: 2026-03-05
**마스터 플랜**: `20260305_P06_fossil_site_evaluator_master_plan.md`

---

## 완료 내역

### P06-1: FossilSite 모델 (fsis 앱 확장)

- `FossilSite` 모델 추가 — site_code 자동 채번 (FS-YYYY-NNNN), 행정위치, WGS84 좌표, 지질정보, 화석정보, 보호현황, 조사이력
- `FossilSitePhoto` 모델 — imagekit 썸네일 (ResizeToFit 400x400)
- `MuseumSpecimen`에 `fossil_site` FK 추가 (nullable, SET_NULL)
- Admin: ImportExport + SimpleHistory + 인라인 사진, list_filter/search_fields
- CRUD 뷰 5개 + 템플릿: fossil_site_list, fossil_site_detail, add/edit/delete
- GeoJSON API: `/fsis/api/fossil-sites/geojson/` (Subquery로 최신 평가 데이터 어노테이션)
- navbar에 화석산지 드롭다운 (산지목록, 산지지도, 위험도 대시보드) 추가

**마이그레이션**: `fsis/migrations/0003_fossilsite_historicalmuseumspecimen_fossil_site_and_more.py`

### P06-2: Evaluator 앱 기반 구축

- `evaluator` 앱 신규 생성, `INSTALLED_APPS` 등록
- `SiteEvaluation` 모델 — Brilha(2016) 5개 항목 위험도 평가
  - fragility(35%), proximity_hazard(20%), legal_protection(20%), accessibility(15%), population_density(10%)
  - risk_score 자동 계산 (가중합), priority 자동 배정 (P0~P4)
  - 보조 데이터: road_distance_m, landcover 5개 항목(%)
- `EvaluationScreenshot` 모델 — 평가 스크린샷 (imagekit 썸네일)
- P0~P4 대시보드 뷰 (우선순위별 통계 카드)
- 평가 CRUD: evaluation_list, evaluation_detail, add/edit/delete_evaluation
- Admin 등록 (스크린샷 인라인)

**마이그레이션**: `evaluator/migrations/0001_initial.py`

### P06-3: gheval 로직 이식

원본: `../gheval` (PyQt6 데스크톱 앱) → Django 서비스 레이어

| 원본 | 이식 위치 | 변경사항 |
|------|----------|----------|
| `GhCommons.py` parse_coordinates | `evaluator/utils/coord_parser.py` | 그대로 이식 |
| `GhCommons.py` fetch_road_distance | `evaluator/services/road_distance.py` | urllib 기반 유지 |
| `GhLandCover.py` analyze_landcover | `evaluator/services/landcover.py` | PyQt6/OpenCV → PIL/numpy |

- Overpass API 도로거리 자동측정 Ajax 엔드포인트: `/evaluator/api/road-distance/<site_pk>/`
- 평가 폼에서 "자동측정" 버튼 → Ajax → 결과 자동 채움
- 좌표 파싱: DMS, DDM, 한국 도분초, decimal 포맷 지원

### P06-4: GIS 지도 플랫폼

- Leaflet.js 1.9.4 + MarkerCluster 기반 인터랙티브 화석산지 지도
- 템플릿: `fsis/templates/fsis/fossil_site_map.html`
- 사이드바 필터: 시도, 보호현황, 우선순위(P0~P4), 검색
- P0~P4별 색상 마커 (빨강→파랑), SVG 아이콘
- 팝업 카드: 산지명, 코드, 위치, 화석군, 위험도 점수
- 위성지도 레이어 옵션 (Esri World Imagery)
- URL: `/fsis/fossil_site_map/`

### P06-5: 표본 DB 고도화

- `SpecimenAssessment` 모델 (evaluator 앱)
  - OneToOne → MuseumSpecimen
  - 완전성(1~3), 보존상태(1~4)
  - 기재표본 여부, 모식표본 종류, 학술적 중요도(1~4)
  - 전시가치(1~3), 교육가치(1~3)
  - overall_grade 자동 계산 (A/B/C/D), grade_color 속성
- 뷰 4개: type_specimen_list, specimen_assessment_detail, add_specimen_assessment, site_specimen_list
- 템플릿 4개: type_specimen_list, specimen_assessment_detail/form, site_specimen_list
- URL 4개: `/evaluator/type-specimens/`, `/evaluator/specimen/<pk>/assessment/`, 등
- navbar에 모식표본 목록 링크 추가

**마이그레이션**: `evaluator/migrations/0002_historicalspecimenassessment_specimenassessment.py`

### P06-6: 공개 플랫폼

일반인·연구자를 위한 공개용 화석지도. 위험도·우선순위 정보 제외.

- `fsis/views_public.py` — 4개 공개 뷰 (인증 불필요)
- 별도 base 템플릿: `fsis/templates/fsis/public/base.html` (독립 디자인, 녹색 테마)

| URL | 뷰 | 설명 |
|-----|-----|------|
| `/public/` | public_map | 공개 화석지도 (Leaflet, 보호현황별 색상 마커) |
| `/public/sites/` | public_site_list | 산지 목록 (검색/필터/페이지네이션) |
| `/public/sites/<pk>/` | public_site_detail | 산지 상세 (지도, 사진 갤러리+라이트박스, kprdb 관련문헌) |
| `/api/public/geojson/` | public_geojson | 공개 GeoJSON API (risk_score, priority 제외) |

- 공개/관리자 데이터 분리: public_geojson은 보호현황·지질시대까지만 노출
- kprdb 관련 문헌 자동 매칭: ReferenceTaxon.location 필드로 산지명/시군구 매칭
- 사진 갤러리: 그리드 레이아웃 + CSS 라이트박스
- SEO: 페이지별 title, meta description 설정
- `config/urls.py`에 공개 URL 4개 등록

---

## 파일 변경 요약

### 신규 파일

```
evaluator/                          # 신규 앱
├── __init__.py
├── apps.py
├── models.py                       # SiteEvaluation, EvaluationScreenshot, SpecimenAssessment
├── admin.py
├── forms.py                        # SiteEvaluationForm, SpecimenAssessmentForm, EvaluationScreenshotForm
├── views.py                        # 대시보드, 평가 CRUD, 표본평가 CRUD, Ajax API
├── urls.py                         # 11개 URL 패턴
├── services/
│   ├── road_distance.py            # Overpass API 도로거리
│   └── landcover.py                # PIL/numpy 토지피복 분석
├── utils/
│   └── coord_parser.py             # 좌표 파싱 (DMS/DDM/Korean/decimal)
├── templates/evaluator/
│   ├── dashboard.html
│   ├── evaluation_list.html
│   ├── evaluation_detail.html
│   ├── evaluation_form.html
│   ├── type_specimen_list.html
│   ├── specimen_assessment_detail.html
│   ├── specimen_assessment_form.html
│   └── site_specimen_list.html
└── migrations/
    ├── 0001_initial.py
    └── 0002_historicalspecimenassessment_specimenassessment.py

fsis/views_public.py                # 공개 플랫폼 뷰
fsis/templates/fsis/public/
├── base.html                       # 공개 base 템플릿
├── map.html                        # 공개 화석지도
├── site_list.html                  # 공개 산지 목록
└── site_detail.html                # 공개 산지 상세
```

### 수정 파일

```
config/settings.py          — evaluator 앱 INSTALLED_APPS 추가
config/urls.py              — evaluator include + 공개 URL 4개 추가
fsis/models.py              — FossilSite, FossilSitePhoto 모델, MuseumSpecimen.fossil_site FK
fsis/admin.py               — FossilSiteAdmin (ImportExport+SimpleHistory+인라인)
fsis/forms.py               — FossilSiteForm, FossilSitePhotoForm
fsis/views.py               — FossilSite CRUD 뷰 6개 + GeoJSON API
fsis/urls.py                — fossil_site 관련 URL 7개
fsis/templates/fsis/
├── fossil_site_list.html
├── fossil_site_detail.html
├── fossil_site_form.html
└── fossil_site_map.html
fsis/templates/fsisnavbar.html — 화석산지 드롭다운 + 모식표본 링크
```

---

## 기술 스택 추가

| 기능 | 라이브러리 | 비고 |
|------|-----------|------|
| 지도 | Leaflet.js 1.9.4 + MarkerCluster 1.5.3 | CDN |
| GeoJSON | Django ORM + Subquery 수동 생성 | 외부 패키지 불필요 |
| 토지피복 분석 | Pillow + numpy | gheval에서 이식 |
| 도로거리 | Overpass API (urllib) | 별도 패키지 불필요 |
