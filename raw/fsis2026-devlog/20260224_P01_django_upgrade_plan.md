# Django 업그레이드 계획 (3.1 → 5.1)

**날짜**: 2026-02-24
**대상**: nkfsite/ (Django 3.1.2)

## 현황 분석

### 좋은 소식
코드베이스가 이미 모던 패턴으로 작성되어 있어 업그레이드 난이도가 **낮음**.

- `path()` 사용 (deprecated `url()` 없음)
- `MIDDLEWARE` 리스트 (구형 `MIDDLEWARE_CLASSES` 아님)
- 모든 `ForeignKey`에 `on_delete` 명시
- `NullBooleanField` 미사용
- deprecated import (`force_text`, `ugettext` 등) 없음
- modern `AppConfig` 사용

### 수정 필요 항목
- `USE_L10N = True` 제거 (Django 4.0에서 deprecated)
- `DEFAULT_AUTO_FIELD` 설정 추가 (Django 3.2 경고)
- 서드파티 패키지 호환성 확인

---

## 서드파티 패키지 현황

| 패키지 | 용도 | Django 5.x 호환 | 비고 |
|--------|------|:---:|------|
| django-simple-history | 변경이력 | O | 활발히 유지보수 |
| django-import-export | Admin import/export | O | 활발히 유지보수 |
| django-autocomplete-light | Select2 자동완성 | O | 활발히 유지보수 |
| django-bootstrap5 | Bootstrap 폼 렌더링 | O | |
| djangorestframework | REST API (미사용) | O | 제거 권장 |
| django-multiselectfield | MultiSelect 필드 | **확인 필요** | 마지막 업데이트 확인 |
| django-archive | 불명 | **확인 필요** | 사용 여부 확인 후 제거 검토 |
| python-decouple | 환경변수 | O | Django 무관 |
| python-hanja, jamo | 한국어 처리 | O | Django 무관 |

---

## 업그레이드 단계

### Step 1: 사전 준비

```bash
# 현재 상태 백업
cp -r nkfsite/ nkfsite_backup/
cp nkfsite/db.sqlite3 nkfsite/db.sqlite3.backup

# Python 버전 확인 (Django 5.x는 Python 3.10+ 필수)
python --version
```

- [ ] git 브랜치 생성: `git checkout -b django-upgrade`
- [ ] 현재 requirements.txt 작성 (없으면 `pip freeze > requirements_old.txt`)
- [ ] 테스트: 현재 상태에서 서버 기동 확인

---

### Step 2: Django 3.1 → 3.2 LTS

**최소 변경, 경고 해결 단계**

```bash
pip install "Django>=3.2,<4.0"
```

수정사항:
- [ ] `settings.py`에 `DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'` 추가
- [ ] `USE_L10N = True` 행 제거
- [ ] `python manage.py check` 실행 → 경고 확인
- [ ] `python manage.py makemigrations` → auto-created primary key 관련 마이그레이션 생성 여부 확인
- [ ] 서버 기동 + 주요 페이지 동작 확인

---

### Step 3: Django 3.2 → 4.2 LTS

**핵심 메이저 업그레이드**

```bash
pip install "Django>=4.2,<5.0"
```

Django 4.0 주요 변경:
- [ ] `USE_L10N` 완전 제거 확인
- [ ] `django.conf.urls` → `django.urls` (이미 호환)
- [ ] `CSRF_TRUSTED_ORIGINS` — 전체 URL 형식 필요 (`https://...`)
- [ ] `password_reset_timeout_days` → `PASSWORD_RESET_TIMEOUT` (미사용 시 무관)

Django 4.1~4.2 변경:
- [ ] `BaseForm.default_renderer` 변경 (폼 렌더링 영향 확인)
- [ ] `python manage.py check --deploy` 실행

서드파티 패키지 업그레이드:
```bash
pip install --upgrade django-simple-history
pip install --upgrade django-import-export
pip install --upgrade django-autocomplete-light
pip install --upgrade django-bootstrap5
```

- [ ] `django-multiselectfield` 호환 확인 → 안되면 대체 방안 검토
- [ ] `django-archive` 사용 여부 확인 → 미사용 시 INSTALLED_APPS에서 제거
- [ ] `djangorestframework` 미사용 확인 → 제거
- [ ] 서버 기동 + 전체 기능 테스트

---

### Step 4: Django 4.2 → 5.1

**최신 버전 (Python 3.10+ 필수)**

```bash
pip install "Django>=5.1,<5.2"
```

Django 5.0 주요 변경:
- [ ] Python 3.10+ 확인
- [ ] `django.utils.encoding.force_str` (이미 미사용이므로 무관)
- [ ] `index_together` → `Meta.indexes` (모델에서 사용 여부 확인)
- [ ] `logout()` 함수 — GET 요청 로그아웃 deprecated → POST로 변경 권장

Django 5.1 변경:
- [ ] 큰 breaking change 없음
- [ ] `python manage.py check` 최종 확인
- [ ] 서버 기동 + 전체 기능 테스트

---

### Step 5: 정리 및 마무리

- [ ] `requirements.txt` 최종 업데이트
- [ ] 불필요 패키지 제거 (`djangorestframework`, `django-archive` 등)
- [ ] `DEBUG = False` 설정 (운영 환경)
- [ ] `ALLOWED_HOSTS` 정리
- [ ] `python manage.py check --deploy` 보안 점검
- [ ] 전체 기능 수동 테스트 (표본 CRUD, 내보내기, 변경이력, 사용자 관리)

---

## 선택사항: 디자인 개선

업그레이드 완료 후 추가 개선 가능 항목:

| 항목 | 설명 | 난이도 |
|------|------|--------|
| Bootstrap 5.3 업데이트 | CDN 버전만 변경 | 낮음 |
| 반응형 개선 | 모바일 레이아웃 점검 | 중간 |
| 다크모드 | Bootstrap 5.3 `data-bs-theme` 활용 | 낮음 |
| 템플릿 정리 | 중복 코드 제거, partial 분리 | 중간 |
| 홈 대시보드 | 통계 요약 페이지 추가 | 중간 |

---

## 위험 요소

| 위험 | 확률 | 대응 |
|------|------|------|
| `django-multiselectfield` 미호환 | 중 | `ArrayField` 또는 JSON 필드로 대체 |
| `django-archive` 미호환 | 중 | 사용 여부 확인 후 제거 |
| 마이그레이션 충돌 | 낮 | `makemigrations --merge` |
| 템플릿 렌더링 차이 | 낮 | Django 4.x 폼 렌더러 변경 확인 |

---

## 예상 소요 시간

| 단계 | 시간 |
|------|------|
| Step 1: 사전 준비 | 30분 |
| Step 2: 3.1 → 3.2 | 1시간 |
| Step 3: 3.2 → 4.2 | 2~3시간 (패키지 호환이 변수) |
| Step 4: 4.2 → 5.1 | 1시간 |
| Step 5: 정리 | 1시간 |
| **합계** | **약 5~6시간** |

대부분의 시간은 서드파티 패키지 호환성 확인과 테스트에 소요됨. 코드 자체 수정은 minimal.
