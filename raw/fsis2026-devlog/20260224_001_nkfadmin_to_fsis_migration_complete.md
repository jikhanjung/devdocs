# nkfadmin → fsis 마이그레이션 및 Django 5.1 프로젝트 구축 완료

**날짜**: 2026-02-24
**계획 문서**: P03 (`devlog/20260224_P03_nkfadmin_to_fsis_migration.md`)

## 작업 요약

원본 Django 3.1.2 프로젝트(`nkfsite/`)의 `nkfadmin` 앱과 `kprdb` 앱을 새로운 Django 5.1 프로젝트(`fsis2026/`)로 마이그레이션했다.

## 수행 단계

### Step 0: 문서화
- `devlog/20260224_P03_nkfadmin_to_fsis_migration.md` 계획 문서 작성
- `docs/HANDOFF.md` 생성

### Step 1: 프로젝트 스캐폴딩
- venv (`/home/jikhanjung/venv/fsis2026/`) 활성화
- Django 5.1.15 설치
- `django-admin startproject config .` 실행
- `.env` 파일 생성 (SECRET_KEY)
- 의존성 설치:
  - django-simple-history, django-import-export, django-autocomplete-light, django-bootstrap5
  - django-multiselectfield, python-decouple
  - hanja, jamo, xlsxwriter, python-docx, Pillow
- 참고: `python-hanja` 패키지는 존재하지 않음 → `hanja`가 올바른 패키지명

### Step 2: 앱 복사
- `nkfsite/nkfadmin/` → `fsis/` 복사
- `nkfsite/kprdb/` → `kprdb/` 복사
- 삭제: `fsis/serializers.py`, 기존 migration 파일, `__pycache__/`

### Step 3: fsis 앱 리네임 (nkfadmin → fsis)
- **apps.py**: `NkfadminConfig` → `FsisConfig`, `name = 'fsis'`
- **urls.py**: DRF import 제거, `app_name = 'fsis'` 추가
- **views.py**:
  - `from nkfsite.settings import MEDIA_ROOT` 삭제
  - DRF import 2줄 삭제, ViewSet 클래스 삭제
  - 모든 템플릿 경로 `'nkfadmin/'` → `'fsis/'` (~30개)
  - 모든 하드코딩 URL redirect → `reverse('fsis:url_name')` (~15개)
- **디렉토리 리네임**:
  - `templates/nkfadmin/` → `templates/fsis/`
  - `static/nkfadmin/` → `static/fsis/`
  - `nkfadmin_data.js` → `fsis_data.js`, `nkfadmin_kakao.html` → `fsis_kakao.html`, `nkfadmin_occurrence.js` → `fsis_occurrence.js`
- **templatetags**: `nkfadmin_extras.py` → `fsis_extras.py`
  - symlink를 통한 `cp -r`이 templatetags 디렉토리를 누락 → 수동 복사로 해결
- **management commands**: `from nkfadmin.models` → `from fsis.models`, 경로 문자열 수정
- **템플릿 38개**: `{% static %}`, `{% url %}`, `{% include %}`, `{% load %}` 태그 전체 수정
- **static 파일 내부**: JS 파일명 참조, 하드코딩 URL 수정

### Step 4: kprdb 앱 조정
- `kprdb/urls.py`에 `app_name = 'kprdb'` 추가
- **views.py**: 36개 `reverse()` 호출에 `'kprdb:'` 접두사 추가, 8개 하드코딩 URL을 `reverse()` 호출로 변환, `LOGIN_URL`에 네임스페이스 추가
- **템플릿 28개**: 105개 `{% url %}` 태그에 `'kprdb:'` 접두사 추가

### Step 5: config/ 설정
- **settings.py**:
  - `SECRET_KEY = config("SECRET_KEY")` (python-decouple)
  - `INSTALLED_APPS`: `fsis`, `kprdb`, `dal`/`dal_select2` (admin보다 앞), `import_export`, `simple_history`, `django_bootstrap5`, `multiselectfield`
  - `simple_history.middleware.HistoryRequestMiddleware` 추가
  - `LANGUAGE_CODE = 'ko-KR'`, `TIME_ZONE = 'Asia/Seoul'`
  - `USE_L10N` 제거 (Django 4.0+에서 deprecated)
  - `DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'`
  - `django_archive`, `djangorestframework` 제거
  - `MEDIA_ROOT`, `MEDIA_URL`, `STATIC_ROOT`, `STATICFILES_DIRS` 설정
- **urls.py**: `fsis/`, `kprdb/`, `admin/` 라우팅 + DEBUG 모드 미디어 서빙

### Step 6: .gitignore 업데이트
- `uploads/` 추가

### Step 7: 마이그레이션 + 검증
- `manage.py check` → 0 issues
- `manage.py makemigrations fsis kprdb` → 초기 마이그레이션 생성
- `manage.py migrate` → 전체 적용 성공
- `manage.py runserver` → 정상 기동

## 변경 파일 수

- 160 files changed, 12,922 insertions(+), 33 deletions(-)

## 발견된 이슈 및 해결

| 이슈 | 해결 |
|------|------|
| `python-hanja` 패키지 없음 | `hanja`로 설치 |
| symlink 통한 `cp -r`이 templatetags 누락 | 절대 경로로 수동 복사 |
| 원본 코드에 `fig_list` URL 없음 | 원본 버그, 현 단계 보존 |
| 원본에 `museum_specimen_photo_form.html` 없음 | 원본 버그, 현 단계 보존 |

## 제거된 의존성

- `djangorestframework` — 미사용 (serializers.py 삭제)
- `django-archive` — Django 5.x 미호환
- `systax` 앱 — 현 단계 미포함

## Git

- 브랜치 `django-upgrade`에서 작업 후 `main`으로 fast-forward merge
- 커밋: `88f378d` "Migrate nkfadmin app to fsis and set up Django 5.1 project"
