# P02: Django 업그레이드 구현 계획

**날짜**: 2026-02-24
**선행 문서**: `20260224_P01_django_upgrade_plan.md`

## 개요

Django 3.1.2 → 5.1 단계별 업그레이드 구현 계획.
Python 3.12.3 환경, venv: `/home/jikhanjung/venv/fsis2026/`

## 수정 대상 파일

| 파일 | 수정 내용 |
|------|----------|
| `nkfsite/nkfsite/settings.py` | DEFAULT_AUTO_FIELD 추가, USE_L10N 제거, STATIC_URL 중복 제거, django_archive 제거 |
| `nkfsite/nkfadmin/views.py:16-17` | 미사용 rest_framework import 제거 |
| `nkfsite/nkfadmin/urls.py:6` | 미사용 rest_framework routers import 제거 |
| `nkfsite/nkfadmin/serializers.py` | 파일 전체 미사용, 제거 |

## 실행 단계

### Step 1: 사전 준비
- 문서화 체계 구축 (CLAUDE.md, docs/HANDOFF.md)
- git 브랜치 `django-upgrade` 생성

### Step 2: Django 3.2 + 코드 수정
- venv에 Django 3.2 + 서드파티 패키지 설치
- settings.py 수정
- 미사용 DRF 코드 정리
- `manage.py check` + `migrate`
- 결과: `devlog/20260224_001_upgrade_to_3.2.md`

### Step 3: Django 4.2
- Django 4.2 업그레이드 + 서드파티 최신화
- `manage.py check`
- 결과: `devlog/20260224_002_upgrade_to_4.2.md`

### Step 4: Django 5.1
- Django 5.1 업그레이드
- django-multiselectfield 호환성 확인
- `manage.py check` + `migrate`
- 결과: `devlog/20260224_003_upgrade_to_5.1.md`

### Step 5: 정리
- requirements.txt 생성
- `manage.py check --deploy`
- 결과: `devlog/20260224_004_final_cleanup.md`

## 검증 방법
1. 각 단계에서 `python manage.py check` 통과
2. 최종 `python manage.py migrate` 성공
3. `python manage.py runserver` 기동 확인
