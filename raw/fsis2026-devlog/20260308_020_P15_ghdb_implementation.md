# P15 ghdb 앱 구현

**날짜**: 2026-03-08

## 배경

2026년 지질유산 데이터베이스 구축 사업 (국가유산청 위탁)을 위한 별도 시스템 구축.
화석·암석 표본의 체계적 DB 구축이 핵심. 참여기관(국공립, 대학, 개인)의 대학원생들이
표본 정보를 입력하는 시스템.

fsis와 유사하나 별도 시스템으로 운영 필요 → 같은 코드베이스, 같은 Docker 이미지,
설정만 분리하여 별도 컨테이너로 운영.

## 작업 내용

### Phase 1: MuseumSpecimen 모델 확장

**커밋**: `33bb260`

#### 1-1. specimen_type 필드 추가 (표본유형 상위 구분)

```python
SPECIMEN_TYPE_CHOICES = [("화석","화석"), ("암석","암석")]
specimen_type = CharField(max_length=20, blank=True, default="화석",
                          choices=SPECIMEN_TYPE_CHOICES, verbose_name='표본유형')
```

기존 `body_or_trace` (체화석/생흔화석)는 화석 내부 구분으로 유지.
`specimen_type`이 상위 구분: 화석 vs 암석.

#### 1-2. 암석 표본 전용 필드 7개 추가

| 필드 | 타입 | 설명 |
|------|------|------|
| `rock_category` | CharField(50) | 암석 대분류 (화성암/퇴적암/변성암) |
| `rock_name` | CharField(200) | 암석명 |
| `rock_name_en` | CharField(200) | 암석명(영문) |
| `rock_texture` | CharField(200) | 조직(texture) |
| `rock_structure` | CharField(200) | 구조(structure) |
| `rock_color` | CharField(100) | 색상 |
| `rock_description` | TextField | 암석 기재 |

모든 필드 `blank=True`이므로 기존 fsis 데이터에 영향 없음.

#### 1-3. 기존 mineral 필드에 verbose_name 추가

`mineral_code1~4`, `mineral_name1~4` 에 `verbose_name` 추가 (광물코드1~4, 광물명1~4).
기존에 verbose_name 없이 정의되어 있었음. UI에서 사용되지 않던 필드.

#### 1-4. source_system 필드 추가 (DB 통합 대비)

**커밋**: `2c18292`

```python
# MuseumSpecimen
source_system = CharField(max_length=10, blank=True, default="fsis", verbose_name='입력시스템')

# UserActivity
source_system = CharField(max_length=10, blank=True, default="fsis")
```

- fsis 기존 데이터: `default="fsis"`로 자동 태깅
- ghdb에서 표본 생성 시: `source_system = "ghdb"` 설정
- 나중에 두 DB를 통합할 때 `WHERE source_system='ghdb'`로 출처 식별 가능

#### 마이그레이션

```
fsis/migrations/0004_historicalmuseumspecimen_rock_category_and_more.py
  - specimen_type, rock_* 7개 필드, mineral verbose_name 변경

fsis/migrations/0005_historicalmuseumspecimen_source_system_and_more.py
  - source_system (MuseumSpecimen, HistoricalMuseumSpecimen, UserActivity)
```

---

### Phase 2: ghdb 앱 생성

**커밋**: `33bb260`

#### 앱 구조

```
ghdb/
├── __init__.py
├── apps.py          # GhdbConfig (verbose_name='지질유산 표본 DB')
├── admin.py
├── models.py        # 비어있음 (fsis.MuseumSpecimen 사용)
├── forms.py         # GhdbSpecimenForm, GhdbSpecimenPhotoForm
├── views.py         # CRUD + 로그인/로그아웃 + UserActivity 기록
├── urls.py          # app_name='ghdb'
├── templates/
│   └── ghdb/
│       ├── base.html            # ghdb 전용 base (fsis 독립)
│       ├── navbar.html          # GHDB 네비게이션바
│       ├── paginator.html       # 페이지네이터
│       ├── specimen_list.html   # 목록 (유형 필터, 정렬)
│       ├── specimen_detail.html # 상세 (화석/암석 조건부 표시, 라이트박스)
│       └── specimen_form.html   # 입력폼 (화석/암석/생흔 JS 토글)
└── migrations/
    └── __init__.py
```

#### forms.py

`GhdbSpecimenForm` — `fsis.MuseumSpecimen` 모델 참조.
fsis의 `MuseumSpecimenForm`과 유사하되:
- `specimen_type` 필드 포함 (상위 구분)
- 암석 전용 필드 (`rock_category`, `rock_name` 등) 포함
- 광물 필드 (`mineral_name1~4`) 포함
- 위젯 설정 (Textarea cols/rows, SelectDateWidget)

`GhdbSpecimenPhotoForm` — fsis의 것과 동일 구조.

#### views.py

| 뷰 | URL | 설명 |
|----|-----|------|
| `specimen_list` | `/` | 목록. filter_type(화석/암석), filter1(검색어), 정렬 |
| `specimen_detail` | `/specimen/<pk>/` | 상세 보기 |
| `specimen_add` | `/specimen/add/` | 새 표본 (`@login_required`) |
| `specimen_edit` | `/specimen/<pk>/edit/` | 편집 (`@login_required`) |
| `specimen_delete` | `/specimen/<pk>/delete/` | 삭제 (`@login_required`) |
| `user_login` | `/login/` | POST 로그인 |
| `user_logout` | `/logout/` | 로그아웃 |

`get_user_obj()` — fsis 버전과 유사하되:
- UserActivity 기록 시 `source_system = "ghdb"` 설정
- `invisible_admin` 예외 처리 없음 (ghdb에서는 불필요)

`specimen_add` — `specimen.source_system = "ghdb"` 설정 후 저장.

#### 템플릿

**base.html**: fsis의 `fsisbase.html`과 독립. 동일 CSS (`geo-theme.css`) 사용하되
`fsis-theme.css` 제외. BabylonJS/Select2 CDN 제외 (불필요).

**navbar.html**: 단순한 구조.
- 표본목록, 화석 필터, 암석 필터
- 로그인/로그아웃 (fsis 시스템 메뉴 제외)

**specimen_list.html**:
- `filter_type` select (전체/화석/암석)
- 유형 컬럼에 badge 표시 (화석=파랑, 암석=회색)
- 표본명 컬럼: 화석이면 `scientific_name`, 암석이면 `rock_name`

**specimen_detail.html**:
- `specimen.specimen_type`에 따라 화석/암석 섹션 조건부 표시 (`{% if %}`)
- 화석일 때: body_or_trace에 따라 체화석/생흔화석 분류 표시
- 암석일 때: 암석 대분류, 암석명, 조직, 구조, 색상, 광물 표시
- 사진 라이트박스 (prev/next 네비게이션)

**specimen_form.html**: 핵심 UI.
- `specimen_type` 선택에 따라 JS로 섹션 토글:
  ```
  specimen_type
  ├── "화석" → fossil_section + fossil_preservation_section 표시
  │   └── body_or_trace
  │       ├── "체화석" → body_table 표시
  │       └── "생흔화석" → trace_table 표시
  └── "암석" → rock_section 표시
  ```
- 사진 formset 동적 추가 (기존 fsis 패턴 동일)

---

### Phase 3: 설정 분리

**커밋**: `33bb260`

#### config/settings_fsis.py

```python
from .settings import *
SITE_TITLE = '지질유산 정보시스템'
```

기존 `settings.py`의 INSTALLED_APPS, ROOT_URLCONF 등을 그대로 상속.
현재는 `settings.py` 자체를 그대로 쓰는 것과 동일.

#### config/settings_ghdb.py

```python
from .settings import *

INSTALLED_APPS = [
    'ghdb.apps.GhdbConfig',
    'fsis.apps.FsisConfig',   # MuseumSpecimen 모델 필요
    # kprdb, evaluator 제외
    ...
]
ROOT_URLCONF = 'config.urls_ghdb'
SITE_TITLE = '지질유산 표본 데이터베이스'
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db_ghdb.sqlite3',
        ...
    }
}
LOGIN_URL = '/login/'
```

- `fsis` 앱 포함 필수 (MuseumSpecimen 모델 참조)
- fsis 전용 테이블(FossilSite, UserActivity 등)도 ghdb DB에 생성됨 — 빈 상태로 무해
- `kprdb`, `evaluator`는 제외

#### config/urls_ghdb.py

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('ghdb.urls')),
]
```

---

## 테스트 결과

- `python manage.py check` — fsis, ghdb 양쪽 모두 이상 없음
- `DJANGO_SETTINGS_MODULE=config.settings_ghdb python manage.py migrate` — ghdb DB 생성 완료
- `DJANGO_SETTINGS_MODULE=config.settings_ghdb python manage.py runserver 8001` — 200 OK
  - `/` (목록) → 200
  - `/specimen/add/` → 302 (로그인 리다이렉트, 정상)

## 로컬 실행 방법

```bash
# fsis (기존)
python manage.py runserver 8000

# ghdb (별도 터미널)
DJANGO_SETTINGS_MODULE=config.settings_ghdb python manage.py runserver 8001
```

## 미완료 (Phase 4: Docker 배포)

계획 문서(`20260308_P15_ghdb_app.md`)에 docker-compose, Nginx, env_file 설정 포함.
실제 배포 시 진행 예정:
- docker-compose.yml에 ghdb 서비스 추가 (같은 이미지, 포트 8001)
- Nginx 프록시 설정 (8001 → ghdb 컨테이너)
- `.env.ghdb` 파일 (별도 SECRET_KEY)
- staticfiles 볼륨 마운트

## 관련 파일

| 파일 | 변경 내용 |
|------|-----------|
| `fsis/models.py` | specimen_type, rock_* 7필드, source_system, mineral verbose_name |
| `fsis/migrations/0004_*` | 암석 필드 + specimen_type 마이그레이션 |
| `fsis/migrations/0005_*` | source_system 마이그레이션 |
| `config/settings_fsis.py` | fsis 전용 설정 (현재는 base 상속만) |
| `config/settings_ghdb.py` | ghdb 전용 설정 (별도 DB, URL, APPS) |
| `config/urls_ghdb.py` | ghdb URL 라우팅 |
| `ghdb/*` | 앱 전체 (forms, views, urls, templates) |
| `devlog/20260308_P15_ghdb_app.md` | 계획 문서 (Phase 1~4) |
