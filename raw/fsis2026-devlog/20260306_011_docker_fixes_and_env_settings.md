# 010: Docker 수정 및 환경변수 분리

## 작업 내용

P12 알려진 이슈 해결, evaluator migration 누락 수정, P13 환경변수 분리 적용.

## 변경 사항

### 1. gunicorn access log 활성화 (P12 이슈)

`entrypoint.sh`에 `--access-logfile -` 옵션 추가.
`docker logs`에서 요청별 access log 확인 가능.

### 2. evaluator migration 누락 수정

- `EvaluationImage.evaluation` 필드의 `related_name` 변경(`'screenshots'` → `'images'`)이 migration에 반영되지 않았음
- 원인: `CLAUDE.md`의 `makemigrations` 명령에 `evaluator` 앱이 빠져 있었음
- `0004_alter_evaluationimage_evaluation.py` 생성 및 적용
- `CLAUDE.md` 수정: `makemigrations fsis kprdb` → `makemigrations fsis kprdb evaluator`

### 3. DEBUG, ALLOWED_HOSTS 환경변수 분리 (P13)

`config/settings.py` 수정:
```python
DEBUG = config("DEBUG", default=False, cast=bool)
ALLOWED_HOSTS = config("ALLOWED_HOSTS", default="localhost,127.0.0.1", cast=lambda v: [s.strip() for s in v.split(',')])
```

개발 환경 `.env`:
```
DEBUG=True
ALLOWED_HOSTS=*
```

운영 서버 `.env`:
```
DEBUG=False
ALLOWED_HOSTS=34.64.158.160,localhost,127.0.0.1
```

### 4. Docker Compose 이미지명 통일

- `docker-compose.yml`, `docker-compose.dev.yml` 모두 `build: .` + `image: honestjung/fsis2026:latest` 지정
- 로컬 빌드 시에도 `honestjung/fsis2026:latest`로 태그됨

### 5. 개발 서버 /srv/fsis2026/ 세팅

운영 서버와 동일한 구조로 개발 서버에도 `/srv/fsis2026/` 디렉토리 생성:
- `db.sqlite3`, `.env`, `uploads/`, `staticfiles/` 배치
- `docker-compose.yml`로 컨테이너 실행 확인

## Docker 이미지 이력

| 버전 | 내용 |
|------|------|
| 0.1.0 | 초기 Docker 이미지 |
| 0.1.1 | gunicorn access log 추가 |
| 0.1.2 | evaluator 0004 migration 포함 |
| 0.1.3 | DEBUG, ALLOWED_HOSTS 환경변수 분리 |
