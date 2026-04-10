# P03: nkfadmin → fsis 앱 마이그레이션 + Django 5.1 신규 프로젝트 구축

## 날짜: 2026-02-24

## 목표
- `nkfsite/nkfadmin` 앱을 `fsis2026/fsis/`로 복사 후 리네임
- `nkfsite/kprdb` 앱을 `fsis2026/kprdb/`로 복사
- Django 5.1 기반 신규 `config/` 프로젝트 구성
- DB 데이터 없음 — 스키마 마이그레이션만

## 환경
- Python 3.12.3
- venv: `/home/jikhanjung/venv/fsis2026/`
- 프로젝트 설정 패키지: `config/`
- `nkfsite/` → readonly 참조용 (심볼릭 링크)

## 작업 단계

### Step 0: 문서화
- 이 계획 문서 생성
- `docs/HANDOFF.md` 갱신
- `CLAUDE.md` 업데이트

### Step 1: 프로젝트 스캐폴딩
- venv + Django 5.1 설치
- `django-admin startproject config .`
- `.env` 생성 (SECRET_KEY)
- 의존성 설치

### Step 2: 앱 복사
- `nkfadmin/` → `fsis/`, `kprdb/` → `kprdb/`
- 불필요 파일 삭제 (serializers, 기존 migrations, __pycache__)

### Step 3: fsis 앱 리네임
- apps.py, urls.py, views.py 수정
- 템플릿/정적 파일 디렉토리 리네임
- 템플릿 내 참조 일괄 수정
- templatetags, management commands 수정

### Step 4: kprdb 앱 조정
- app_name 추가, URL 네임스페이스 적용

### Step 5: config/ 설정
- settings.py, urls.py 작성

### Step 6-7: gitignore + 마이그레이션 + 검증

## 리스크
- django-multiselectfield: Django 5.x 미호환 가능
- URL name 충돌: app_name 네임스페이스로 해결
- 기존 버그 보존: fig_list URL 없음, museum_specimen_photo_form.html 없음
