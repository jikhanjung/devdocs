# 027. 배포 자동화 및 일괄 입력 기능 개선 (2026-03-22)

## 배포 인프라 정비

### Docker Compose 분리
- 기존 `deploy/docker-compose.yml` 단일 파일에서 서비스별 분리
  - `/srv/fsis2026/docker-compose.yml` — fsis 전용
  - `/srv/ghdb/docker-compose.yml` — ghdb 전용
- 이미지 태그를 `${IMAGE_TAG}` 환경변수로 관리

### 환경 파일 분리
- `.env` → compose 배포용 (`IMAGE_TAG` 등)
- `.env.django` → Django 앱 환경변수 (`SECRET_KEY`, `DEBUG` 등)
- compose의 `env_file` 지시어를 `.env.django`로 변경

### 배포 스크립트
- `/srv/fsis2026/deploy.sh`, `/srv/ghdb/deploy.sh` 생성
- 사용법: `deploy.sh <version>` → .env 수정 → pull → down → up
- deploy 계정으로 SSH 원격 배포 가능: `ssh dolfinid-deploy "/srv/fsis2026/deploy.sh 0.2.13"`

## fsis 일괄 입력 버그 수정

### 로그인 리다이렉트 500 에러
- `@login_required(login_url='fsis:fsis_user_login')` → `fsis_user_login`이 POST만 처리
- `login_url='/fsis/'`로 변경하여 메인 페이지(navbar 로그인 폼)로 리다이렉트

### navbar 로그인 상태 미표시
- `views_import.py`의 뷰들에서 `user_obj`를 템플릿에 전달하지 않던 문제
- `get_user_obj(request)` 호출 및 모든 render context에 `user_obj` 추가

### 날짜 파싱 오류
- openpyxl이 날짜 셀을 `datetime` 객체로 반환 → `str()` 변환 시 `"2026-03-15 00:00:00"` 형태
- `_parse_date()`에서 `datetime`/`date` 인스턴스 직접 처리하도록 수정

### 오류 행 미표시
- `ok_count == 0`일 때 테이블 자체가 숨겨지던 문제
- 오류만 있어도 테이블은 표시하고, 저장 버튼만 조건부 표시로 변경

## ghdb 일괄 입력 기능 추가

- 체화석 / 생흔화석 / 암석 3가지 양식 지원
- `ghdb/import_utils.py` — 암석 전용 컬럼 (rock_*, mineral_*) 포함
- `ghdb/views_import.py`, 템플릿 2개, URL 3개 추가
- navbar에 일괄입력 링크 추가
- `source_system='ghdb'`로 저장

## UI 개선

### fsis 브랜딩 변경
- 페이지 타이틀: `KOFHIN — Korea Fossil Heritage Inventory`
- navbar 브랜드: `KOFHIN`

### 버튼 글자색 수정
- fsis-theme에서 `btn-geo-primary`에 `color: #fff` 누락 → 추가
- `.fsis-theme a` 링크 색상이 `.btn` 클래스에도 적용되던 문제 → `a:not(.btn)`으로 제한

## 버전 이력

| 버전 | 내용 |
|------|------|
| 0.2.6 | bulk import 기능 포함 첫 빌드 |
| 0.2.7 | 로그인 리다이렉트, navbar user_obj 수정 |
| 0.2.8 | 오류 행 테이블 표시 수정 |
| 0.2.9 | 날짜 파싱 (datetime 객체) 수정 |
| 0.2.10 | ghdb 일괄 입력 기능 추가 |
| 0.2.11 | KOFHIN 브랜딩 |
| 0.2.12 | 버튼 글자색 수정 (btn-geo-primary) |
| 0.2.13 | 버튼 링크 색상 수정 (a:not(.btn)) |
