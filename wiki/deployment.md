# 배포 인프라

FSIS2026와 GHDB의 운영 환경 구성.

## 서버 환경

- **서버**: GCP 34.64.158.160 (DolfinServer)
- **OS**: Linux
- **도메인**: geolmapkr.nopeoplestime.info
- **SSL**: Let's Encrypt certbot (FSIS: 표준 포트, GHDB: 8001 수동 설정)

## Docker 구성

- **베이스 이미지**: python:3.12-slim
- **워커**: Gunicorn 3 workers
- **이미지 레지스트리**: Docker Hub (honestjung/fsis2026)
- **버전 관리**: config/version.py + context processor

### 볼륨 마운트 (FHS /srv 표준)
```
/srv/fsis2026/
├── db.sqlite3      # SQLite DB
├── uploads/        # 미디어 파일 (8.3GB)
├── staticfiles/    # collectstatic 출력 (553MB)
└── .env            # 환경변수
```

### Docker 파일 구조
```
deploy/
├── Dockerfile
├── entrypoint.sh    # collectstatic + migrate 자동 실행
├── docker-compose.yml
└── docker-compose.dev.yml
```

### 이미지 크기 최적화
- .dockerignore: 550MB map_tiles, testdata 제외
- 1.56GB → 473MB 축소 (v0.2.3)

## Nginx 설정

- TCP 프록시 → Docker 컨테이너 (포트 8000)
- map_tiles: 별도 location으로 직접 서빙
- static/media: 정적 파일 서빙 + Django fallback
- legacy Apache 정적 파일: /var/www/html/ fallback
- max_body_size: 100M (대용량 PDF 업로드)
- HTTP → HTTPS 리다이렉트

## 환경변수 (python-decouple)

| 변수 | 용도 |
|---|---|
| DEBUG | True/False (운영: False) |
| ALLOWED_HOSTS | 쉼표 구분 호스트 목록 |
| SECRET_KEY | Django 시크릿 ($ 문자 제한 주의) |
| DATABASE_PATH | DB 경로 분기 |
| MEDIA_ROOT | 미디어 경로 분기 |

## 보안 설정

Django check --deploy 기반:
- SECURE_SSL_REDIRECT, SESSION_COOKIE_SECURE, CSRF_COOKIE_SECURE
- SECURE_HSTS_SECONDS (초기 1h)
- SECURE_PROXY_SSL_HEADER (nginx 프록시용)
- 모두 환경변수 게이팅 (기본 False/0)

## SQLite 동시성

- timeout: 5초 → 20초
- Django 5.1 transaction_mode='IMMEDIATE'
- WAL 모드: 읽기/쓰기 블로킹 제거 (5-10 동시 사용자)

## 백업 시스템

**외부 pull 방식** (백업 서버에서 운영 서버로 SSH+rsync)

### 스케줄
| 작업 | cron |
|---|---|
| DolfinServer DB 백업 | 매일 01:00 |
| 외부 rsync pull | 매일 03:00 |

### 보존 정책
- 로컬: 30일 daily → monthly
- NAS: 90일 daily → monthly
- 연간 아카이브: 12월 1일

### 경로
- 로컬: /home/jikhanjung/backups/fsis2026/
- NAS: /nas/JikhanJung/fsis2026_backup/
- WAL 파일 백업 포함

## 배포 자동화

- **build.sh**: test → version bump → docker build → push
- **deploy.sh**: 자동 버전 업데이트 스크립트
- 버전 추적: v0.1.0 → v0.3.61+ (2026-04-10 기준)

## OCR cron 자동화

- ocr_cron.sh: 10분 주기 PDF OCR 처리
- claude_augment cron: 30분 주기 Claude 보강
- lockfile 안전장치, umask 002 (그룹 쓰기 권한)
- /srv/fsis2026/venv 가상환경 (PyMuPDF, Pillow, requests)

## 관련 페이지

- [GHDB](ghdb.md) — GHDB 배포 세부사항
- [PDF 파이프라인](pdf-pipeline.md) — OCR cron 연계
- [프로젝트 개요](overview.md)

---
*Sources: 002, 010, 011, 015, 016, 017, 021, 022, 023, 024, 025, 026, 027, 028, 052, 060, P10, P11, P12, P13, P16*
