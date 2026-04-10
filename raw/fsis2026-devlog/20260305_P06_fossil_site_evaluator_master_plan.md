# P06: 화석산지 평가 시스템 마스터 플랜

**작성일**: 2026-03-05
**기반 문서**: `data/proposal20260223.pdf` (국내 주요 화석산지 발굴 및 활용 중장기계획 수립)
**참조 프로젝트**: `../gheval` (GeoHeritage Evaluator PyQt6 데스크톱 앱)

---

## 배경 및 목표

국가유산청 발주 과제: 전국 주요 화석산지를 체계적으로 조사·평가하고, 발굴 우선순위를 선정하여 중장기 보전·활용 계획을 수립한다.

**핵심 산출물**:
- 전국 화석산지 목록 및 GIS DB
- 위험도 기반 발굴 우선순위 (P0~P4)
- 모식표본 및 주요 표본 관리 DB 고도화
- 공개용 화석지도 웹 플랫폼

---

## 아키텍처 결정 사항

| 구분 | 결정 | 이유 |
|------|------|------|
| `fsis` 앱 | `FossilSite` 모델 추가 | 기존 Location 대체, MuseumSpecimen과 연동 |
| `kprdb` 앱 | `ReferenceTaxonSpecimen`에 FK 추가만 | 최소 수정 원칙 |
| 신규 `evaluator` 앱 | 완전 신규 생성 | 위험도 평가 도메인 분리 |
| gheval 로직 | Python 서비스 레이어로 이식 | UI(PyQt6) 제외, 알고리즘만 재사용 |

---

## Phase 개요

| Phase | 제목 | 주요 작업 | 선행 조건 |
|-------|------|----------|----------|
| P06-1 | FossilSite 모델 | fsis 앱 FossilSite 모델 추가 | 없음 |
| P06-2 | Evaluator 앱 기반 | evaluator 앱 생성, 위험도 모델 | P06-1 |
| P06-3 | gheval 로직 이식 | 도로거리, 토지피복, 좌표파싱 | P06-2 |
| P06-4 | GIS 지도 플랫폼 | Leaflet 기반 웹 지도, 클러스터 | P06-1 |
| P06-5 | 표본 DB 고도화 | Specimen-ID, 모식표본 가치평가 | P06-1 |
| P06-6 | 공개 플랫폼 | 공개용 화석지도, API 엔드포인트 | P06-4, P06-5 |

---

## Phase 1: FossilSite 모델 (`fsis` 앱 확장)

### 목표
전국 화석산지 기본 정보를 체계적으로 관리하는 Django 모델 구축

### 신규 모델: `FossilSite`

```python
class FossilSite(models.Model):
    # === 기본 식별 ===
    site_code = models.CharField(max_length=20, unique=True)  # FS-2026-0001
    site_name = models.CharField(max_length=200)
    site_name_en = models.CharField(max_length=200, blank=True)

    # === 행정 위치 ===
    province = models.CharField(max_length=50)   # 시도
    city = models.CharField(max_length=50)        # 시군구
    district = models.CharField(max_length=100, blank=True)  # 읍면동
    address = models.TextField(blank=True)

    # === 좌표 (WGS84) ===
    latitude = models.FloatField()
    longitude = models.FloatField()
    coord_accuracy = models.CharField(max_length=20, default='GPS')  # GPS/Map/Est

    # === 지질 정보 ===
    geologic_age = models.ForeignKey('ChronoUnit', null=True, blank=True, on_delete=models.SET_NULL)
    litho_unit = models.ForeignKey('LithoUnit', null=True, blank=True, on_delete=models.SET_NULL)
    rock_type = models.CharField(max_length=100, blank=True)

    # === 화석 정보 ===
    fossil_group = models.CharField(max_length=200, blank=True)  # 식물/어류/공룡 등
    fossil_desc = models.TextField(blank=True)
    significance = models.TextField(blank=True)  # 학술적 가치

    # === 보호 현황 ===
    PROTECTION_CHOICES = [
        ('천연기념물', '천연기념물'),
        ('문화재자료', '문화재자료'),
        ('지질공원', '지질공원'),
        ('국립공원', '국립공원'),
        ('무보호', '무보호'),
        ('미확인', '미확인'),
    ]
    protection_status = models.CharField(max_length=20, choices=PROTECTION_CHOICES, default='미확인')
    protection_number = models.CharField(max_length=50, blank=True)  # 지정 번호

    # === 조사 이력 ===
    first_reported_year = models.IntegerField(null=True, blank=True)
    last_surveyed = models.DateField(null=True, blank=True)
    survey_count = models.IntegerField(default=0)

    # === 메타 ===
    notes = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    history = HistoricalRecords()

    class Meta:
        ordering = ['site_code']

    def __str__(self):
        return f"[{self.site_code}] {self.site_name}"
```

### 기존 모델 연동

```python
# fsis/models.py - MuseumSpecimen에 추가
fossil_site = models.ForeignKey(
    'FossilSite', null=True, blank=True,
    on_delete=models.SET_NULL, related_name='specimens'
)

# kprdb/models.py - ReferenceTaxonSpecimen에 추가
fossil_site = models.ForeignKey(
    'fsis.FossilSite', null=True, blank=True,
    on_delete=models.SET_NULL, related_name='kprdb_specimens'
)
```

### 작업 목록
- [ ] `FossilSite` 모델 작성 및 마이그레이션
- [ ] `FossilSitePhoto` 모델 (imagekit 썸네일 포함)
- [ ] Admin 등록 (list_display, search_fields, 지도 미리보기)
- [ ] `MuseumSpecimen` → `FossilSite` FK 추가 및 마이그레이션
- [ ] `ReferenceTaxonSpecimen` → `FossilSite` FK 추가 및 마이그레이션
- [ ] site_code 자동 채번 로직 (FS-YYYY-NNNN)
- [ ] 기존 Location 데이터에서 FossilSite 변환 스크립트

### 산출물
- `fsis/models.py`: FossilSite, FossilSitePhoto 모델
- `fsis/migrations/XXXX_add_fossilsite.py`
- `fsis/admin.py`: FossilSite 관리자 인터페이스

---

## Phase 2: Evaluator 앱 기반 구축

### 목표
Brilha(2016) 기반 5개 항목 위험도 평가 모델 + P0~P4 우선순위 시스템

### 위험도 평가 항목 (Brilha 2016)

| 항목 | 코드 | 가중치 | 측정 방법 |
|------|------|--------|----------|
| 훼손가능성 | `fragility` | 35% | 암석 강도, 노출도, 접근성 |
| 근접 위험 활동 | `proximity_hazard` | 20% | 개발지, 채석장, 도로 공사 반경 |
| 법적 보호 상태 | `legal_protection` | 20% | 미보호=5, 지자체=3, 국가=1 |
| 접근성 | `accessibility` | 15% | 포장도로까지 거리 (역방향: 가까울수록 위험) |
| 인구 밀도 | `population_density` | 10% | 반경 10km 내 인구 |

**총점 계산**: Σ(점수 × 가중치) / 5 → 1.0 ~ 5.0

**P0~P4 우선순위**:
- P0: 위험도 4.0~5.0, 즉시 긴급 조치 필요
- P1: 위험도 3.0~3.9, 단기 계획 (1~3년)
- P2: 위험도 2.0~2.9, 중기 계획 (3~5년)
- P3: 위험도 1.0~1.9, 장기 계획 (5년+)
- P4: 위험도 <1.0 or 이미 보호됨

### 신규 앱: `evaluator`

#### 모델 구조

```python
# evaluator/models.py

class SiteEvaluation(models.Model):
    """화석산지 위험도 평가 (1회 평가 = 1 레코드)"""
    fossil_site = models.ForeignKey('fsis.FossilSite', on_delete=models.CASCADE,
                                     related_name='evaluations')
    evaluator = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL, null=True)
    eval_date = models.DateField()

    # === Brilha 5개 항목 (각 1~5점) ===
    fragility = models.FloatField()            # 훼손가능성 (35%)
    proximity_hazard = models.FloatField()     # 근접 위험 활동 (20%)
    legal_protection = models.FloatField()     # 법적 보호 상태 (20%)
    accessibility = models.FloatField()        # 접근성 (15%)
    population_density = models.FloatField()  # 인구 밀도 (10%)

    # === 자동 계산 ===
    risk_score = models.FloatField(editable=False)  # 가중합
    priority = models.CharField(max_length=2, editable=False)  # P0~P4

    # === 보조 데이터 ===
    road_distance_m = models.FloatField(null=True, blank=True)  # Overpass API
    landcover_dense_veg = models.IntegerField(null=True, blank=True)  # %
    landcover_sparse_veg = models.IntegerField(null=True, blank=True)
    landcover_bare = models.IntegerField(null=True, blank=True)
    landcover_built = models.IntegerField(null=True, blank=True)
    landcover_water = models.IntegerField(null=True, blank=True)
    landcover_radius_m = models.IntegerField(default=500)

    notes = models.TextField(blank=True)
    history = HistoricalRecords()

    def save(self, *args, **kwargs):
        self.risk_score = self._calculate_risk()
        self.priority = self._calculate_priority()
        super().save(*args, **kwargs)

    def _calculate_risk(self):
        weights = [0.35, 0.20, 0.20, 0.15, 0.10]
        scores = [self.fragility, self.proximity_hazard, self.legal_protection,
                  self.accessibility, self.population_density]
        return sum(s * w for s, w in zip(scores, weights))

    def _calculate_priority(self):
        s = self.risk_score
        if s >= 4.0: return 'P0'
        if s >= 3.0: return 'P1'
        if s >= 2.0: return 'P2'
        if s >= 1.0: return 'P3'
        return 'P4'


class EvaluationScreenshot(models.Model):
    """평가 시 캡처한 위성/지도 이미지"""
    evaluation = models.ForeignKey(SiteEvaluation, on_delete=models.CASCADE,
                                    related_name='screenshots')
    image = ProcessedImageField(...)  # imagekit
    map_type = models.CharField(max_length=20)  # HYBRID/ROADMAP/SKYVIEW
    zoom_level = models.IntegerField(default=15)
    wayback_date = models.CharField(max_length=20, blank=True)  # ESRI Wayback
    captured_at = models.DateTimeField(auto_now_add=True)
```

### 작업 목록
- [ ] `evaluator` 앱 생성 및 `INSTALLED_APPS` 등록
- [ ] `SiteEvaluation`, `EvaluationScreenshot` 모델 작성
- [ ] `evaluator/services.py`: 위험도 계산 순수 함수들
- [ ] Admin 인터페이스: 5개 항목 입력 폼, 자동 점수 표시
- [ ] P0~P4 대시보드 뷰 (우선순위별 목록)
- [ ] 평가 히스토리 관리 (simple_history)

### 산출물
- `evaluator/` 앱 디렉토리 전체
- `evaluator/models.py`, `evaluator/admin.py`, `evaluator/views.py`
- `evaluator/services.py`: 위험도 계산 로직
- P0~P4 우선순위 대시보드 템플릿

---

## Phase 3: gheval 로직 이식

### 목표
gheval 데스크톱 앱의 핵심 알고리즘을 Django 서비스로 이식

### 재사용 가능 컴포넌트

| gheval 파일 | 재사용 함수 | Django 이식 위치 |
|-------------|------------|----------------|
| `GhCommons.py` | `parse_coordinates()`, `scan_coordinates_in_text()` | `evaluator/utils/coord_parser.py` |
| `GhCommons.py` | `fetch_road_distance()` | `evaluator/services/road_distance.py` |
| `GhCommons.py` | `road_distance_to_score()` | `evaluator/services/road_distance.py` |
| `GhLandCover.py` | `analyze_landcover()` | `evaluator/services/landcover.py` |
| `GhCommons.py` | ESRI Wayback 탐색 로직 | `evaluator/services/wayback.py` |
| `GhPdfExtractor.py` | 텍스트 좌표 추출 | `evaluator/management/commands/` |

### 새로 작성할 부분

#### `evaluator/services/road_distance.py`
```python
"""
gheval GhCommons.fetch_road_distance() 이식
Overpass API로 가장 가까운 포장도로까지 거리 계산
"""
import urllib.request, json, math

OVERPASS_URL = "https://overpass-api.de/api/interpreter"

ROAD_TYPES = [
    'trunk', 'trunk_link',
    'primary', 'primary_link',
    'secondary', 'secondary_link',
    'tertiary', 'tertiary_link',
]

def fetch_road_distance(lat, lon, radius_m=5000):
    """Overpass API로 nearest road distance (m) 반환. 실패 시 None."""
    ...

def road_distance_to_score(distance_m):
    """거리 → 위험도 점수 (1~5).
    <500m=5, 500-1000m=4, 1-2km=3, 2-5km=2, >5km=1
    """
    ...
```

#### `evaluator/services/landcover.py`
```python
"""
gheval GhLandCover.analyze_landcover() 이식
위성사진 → HSV+ExG → 토지피복 분류 (%)
PIL 기반으로 대체 (PyQt6 의존성 제거)
"""
from PIL import Image
import numpy as np

def analyze_landcover_from_url(lat, lon, zoom=15, radius_m=500):
    """위성 타일 다운로드 후 토지피복 분석"""
    ...

def analyze_landcover_from_image(image_path, radius_m=500):
    """저장된 이미지 파일에서 토지피복 분석"""
    ...
```

### 핵심 변경사항 (gheval → Django)

| 항목 | gheval | Django 이식 |
|------|--------|------------|
| 이미지 처리 | QPixmap (PyQt6) | PIL/Pillow |
| 비동기 처리 | QThread | Celery task 또는 동기 |
| 결과 저장 | Peewee → SQLite | Django ORM → SQLite3 |
| 좌표 파싱 | 독립 함수 | 그대로 이식 가능 |

### 작업 목록
- [ ] `evaluator/utils/coord_parser.py`: GhCommons parse 함수 이식
- [ ] `evaluator/services/road_distance.py`: Overpass API 연동
- [ ] `evaluator/services/landcover.py`: PIL 기반 토지피복 분석
- [ ] `evaluator/services/wayback.py`: ESRI Wayback 여름영상 탐색
- [ ] `evaluator/tasks.py`: Celery task (비동기 도로거리+토지피복)
- [ ] 평가 폼에서 "자동 분석" 버튼 → Ajax 요청 → 결과 채우기
- [ ] 관리 명령어: `import_from_pdf` (GhPdfExtractor 이식)

### 산출물
- `evaluator/utils/`, `evaluator/services/` 디렉토리
- 자동 분석 기능이 포함된 평가 입력 폼
- `management/commands/import_sites_from_pdf.py`

---

## Phase 4: GIS 지도 플랫폼

### 목표
Leaflet.js 기반 인터랙티브 화석산지 지도

### 뷰 설계

#### 메인 지도 (`/fsis/map/fossils/`)
```
┌────────────────────────────────────────────────────┐
│ [필터: 도/시] [우선순위 P0~P4] [지질시대] [보호상태]  │
├──────────┬─────────────────────────────────────────┤
│ 사이드패널│  Leaflet Map                             │
│          │  ┌──────────────────────────────────┐   │
│ 📍 FS-001│  │  ● ● ●  클러스터 마커              │   │
│ 공룡발자국│  │     ↓ 클릭                         │   │
│ P1 | 전남│  │  ┌─────────────────────────┐      │   │
│ ──────── │  │  │ FS-001 공룡발자국화석지  │      │   │
│ 📍 FS-002│  │  │ 위험도: ████░ 3.4 (P1)  │      │   │
│          │  │  │ 전남 화순군 | 백악기     │      │   │
│          │  │  │ [상세보기] [평가이력]    │      │   │
│          │  │  └─────────────────────────┘      │   │
│          │  └──────────────────────────────────┘   │
└──────────┴─────────────────────────────────────────┘
```

#### GeoJSON API 엔드포인트
```
GET /fsis/api/fossil-sites/geojson/
  ?priority=P0,P1
  &province=전라남도
  &age=백악기
  → GeoJSON FeatureCollection

GET /fsis/api/fossil-sites/{id}/
  → 단일 산지 상세 JSON
```

### 마커 스타일 (P0~P4별 색상)
- P0: 빨강 🔴, P1: 주황 🟠, P2: 노랑 🟡, P3: 초록 🟢, P4: 파랑 🔵, 미평가: 회색 ⚪

### 작업 목록
- [ ] `fsis/views.py`: 지도 뷰, GeoJSON API 뷰
- [ ] `fsis/urls.py`: `/map/fossils/`, `/api/fossil-sites/` 등록
- [ ] Leaflet.js + MarkerCluster 템플릿
- [ ] P0~P4 색상 마커, 팝업 카드
- [ ] 필터 패널 (도/시, 우선순위, 지질시대, 보호상태)
- [ ] 사이드 패널: 필터링된 목록 표시
- [ ] 관리자용 지도: 평가 현황 통계 오버레이

### 산출물
- `fsis/templates/fsis/fossil_map.html`
- `fsis/templates/fsis/fossil_map_data.html` (API 포함)
- Leaflet 기반 인터랙티브 지도

---

## Phase 5: 표본 DB 고도화

### 목표
모식표본 가치평가 체계 및 표본-산지 연동 강화

### 모식표본 가치평가 항목

```python
class SpecimenAssessment(models.Model):
    """MuseumSpecimen에 대한 학술적 가치평가"""
    specimen = models.OneToOneField('fsis.MuseumSpecimen', on_delete=models.CASCADE)

    # === 완전성 ===
    completeness = models.IntegerField(choices=[(1,'단편'),(2,'부분'),(3,'완전')])
    preservation = models.IntegerField(choices=[(1,'불량'),(2,'보통'),(3,'양호'),(4,'우수')])

    # === 학술 가치 ===
    is_type_specimen = models.BooleanField(default=False)  # 기재표본 여부
    nomenclatural_status = models.CharField(...)  # 모식표본 종류
    scientific_priority = models.IntegerField(choices=[(1,'낮음'),(2,'보통'),(3,'높음'),(4,'매우높음')])

    # === 전시/활용 가치 ===
    display_value = models.IntegerField(choices=[(1,'낮음'),(2,'보통'),(3,'높음')])
    educational_value = models.IntegerField(choices=[(1,'낮음'),(2,'보통'),(3,'높음')])

    # === 종합 등급 ===
    overall_grade = models.CharField(max_length=1, editable=False)  # A/B/C/D
    notes = models.TextField(blank=True)
```

### 작업 목록
- [ ] `SpecimenAssessment` 모델 추가 (evaluator 앱 또는 fsis 앱)
- [ ] Specimen-ID 체계 정리 (FS-YYYY-NNNN-S-NNN)
- [ ] 표본 목록 → 산지 연동 뷰 (어느 산지에서 채집됐는지)
- [ ] 모식표본 전용 목록/검색 페이지
- [ ] kprdb ReferenceTaxonSpecimen → FossilSite 연동 뷰

### 산출물
- `SpecimenAssessment` 모델
- 모식표본 가치평가 입력/조회 페이지
- 표본-산지 연동 통계 페이지

---

## Phase 6: 공개 플랫폼

### 목표
일반인·연구자를 위한 공개용 화석지도 및 정보 조회

### 설계 원칙
- 위험도·우선순위 정보는 **관리자 전용** (공개 불가)
- 공개 정보: 산지명, 행정구역, 지질시대, 화석종류, 보호상태, 대표사진
- 관련 논문(kprdb 참조) 링크 제공

### 공개 URL 구조
```
/public/                    → 공개 화석지도 메인
/public/sites/              → 산지 목록 (검색/필터)
/public/sites/<id>/         → 산지 상세 (공개 정보만)
/public/taxa/               → 화석종류별 검색
/api/public/geojson/        → 공개 GeoJSON API
```

### 작업 목록
- [ ] 공개용 URL 라우팅 및 퍼미션 분리
- [ ] 공개 지도 템플릿 (관리 지도와 별도)
- [ ] 산지 상세 공개 페이지 (사진 갤러리 포함)
- [ ] kprdb 참고문헌 연동 표시
- [ ] 공개 GeoJSON API (민감정보 제외)
- [ ] SEO 최적화 (메타 태그, sitemap.xml)

### 산출물
- `fsis/views_public.py`
- `fsis/templates/fsis/public/` 디렉토리
- 공개용 화석지도 완성

---

## 의존 관계 다이어그램

```
P06-1 (FossilSite 모델)
    ├── P06-2 (Evaluator 앱) ← P06-3 (gheval 이식)
    ├── P06-4 (GIS 지도)
    └── P06-5 (표본 DB)
            ↓
        P06-6 (공개 플랫폼)
```

P06-4와 P06-5는 P06-2와 병렬 진행 가능.

---

## 기술 스택 추가사항

| 기능 | 라이브러리 | 비고 |
|------|-----------|------|
| 지도 | Leaflet.js + MarkerCluster | CDN 사용 |
| 토지피복 분석 | Pillow + numpy | requirements.txt 추가 |
| Overpass API | 표준 urllib | 별도 패키지 불필요 |
| 비동기 작업 | Celery + Redis | 선택사항 (동기 fallback 가능) |
| GeoJSON 생성 | django-geojson 또는 수동 | 단순 수동 구현으로 충분 |

---

## 데이터 흐름

```
PDF 보고서
    → GhPdfExtractor → 좌표 추출
        → FossilSite 등록 (관리자)
            → SiteEvaluation 생성
                → Overpass API → road_distance → accessibility 점수
                → 위성사진 → landcover_analyze → fragility 보조 데이터
                → 관리자 5개 항목 입력
                    → risk_score 자동 계산
                        → P0~P4 우선순위 자동 배정
                            → GIS 지도에 시각화
```

---

## 예상 파일 구조 (완료 후)

```
fsis2026/
├── fsis/
│   ├── models.py          ← FossilSite, FossilSitePhoto 추가
│   ├── views.py           ← 지도 뷰, GeoJSON API
│   ├── templates/fsis/
│   │   ├── fossil_map.html         ← P06-4
│   │   └── public/                 ← P06-6
│   └── ...
├── kprdb/
│   └── models.py          ← ReferenceTaxonSpecimen FK 추가
└── evaluator/             ← P06-2~3 신규
    ├── __init__.py
    ├── apps.py
    ├── models.py
    ├── admin.py
    ├── views.py
    ├── urls.py
    ├── services/
    │   ├── road_distance.py
    │   ├── landcover.py
    │   └── wayback.py
    ├── utils/
    │   └── coord_parser.py
    ├── tasks.py
    ├── templates/evaluator/
    └── management/commands/
        └── import_sites_from_pdf.py
```
