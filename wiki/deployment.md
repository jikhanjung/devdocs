# 배포 인프라

FSIS2026 / [KOFHIN](fsis2026-overview.md)과 [GHDB](ghdb.md)의 운영 환경 구성. 별도의 `docs/server_architecture.md` 문서(2026-04-07 기준)가 현재 배포 상태의 authoritative 소스다.

## 전체 구조

```
프로덕션 서버 (34.64.158.160, GCP DolfinServer)
│
├── Nginx (리버스 프록시, SSL)
│   ├── :443 (HTTPS) → fsis 컨테이너 :8000
│   └── :8001 (SSL)  → ghdb 컨테이너 :8002
│
├── Docker 컨테이너
│   ├── fsis (config.settings_fsis) — 메인 앱 (KOFHIN)
│   └── ghdb (config.settings_ghdb) — 지질유산 DB
│
├── 호스트 서비스 (Docker 밖)
│   ├── cron (OCR + 정규식 추출 + Claude 보강, 10분 간격)
│   └── 서버 전용 venv (/srv/fsis2026/venv/)
│
└── 공유 볼륨
    ├── /srv/fsis2026/db.sqlite3
    ├── /srv/fsis2026/uploads/references/ (PDF, OCR JSON, 추출 JSON)
    └── /srv/fsis2026/staticfiles/
```

## 서버 환경

- **서버**: GCP 34.64.158.160 (DolfinServer)
- **OS**: Linux
- **프로덕션 URL**: **https://fsis.psok.or.kr** (KOFHIN 공식)
- **레거시 도메인**: geolmapkr.nopeoplestime.info
- **SSL**: Let's Encrypt certbot (FSIS: 표준 포트, GHDB: 8001 수동 설정)
- **현재 이미지 태그**: `IMAGE_TAG=0.3.16` (2026-04-07 기준)

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

## 호스트 OCR/추출 파이프라인 (Docker 밖)

### 왜 Docker 밖에서 실행하는가

- **RunPod API 키 격리**: `/srv/fsis2026/scripts/.env`에만 존재, Docker 이미지에 포함하지 않음
- **최소 의존성 venv**: `/srv/fsis2026/venv/` — OCR에 필요한 최소 패키지만 (PyMuPDF, requests, python-dotenv, Pillow)
- **독립 실행**: Django 의존 없이 스크립트 단독 실행 가능
- Docker 컨테이너는 **웹 서빙에만 집중**

### cron 구성

```cron
# OCR (증분, .last_ocr_run 기반)
*/10 * * * * /srv/fsis2026/scripts/ocr_cron.sh >> /srv/fsis2026/scripts/ocr_cron.log 2>&1

# Claude 보강 (1개씩, 오래된 것부터)
*/10 * * * * /srv/fsis2026/venv/bin/python /srv/fsis2026/scripts/claude_augment.py --limit 1 >> /srv/fsis2026/scripts/claude_augment.log 2>&1
```

### 파이프라인 흐름

```
cron 실행 (10분 간격)
├── OCR: find → batch_runpod.py → {id}.pdf.json
├── 정규식 추출: extract_from_ocr.py → {id}.pdf.extract.json
└── Claude 보강: claude_augment.py → {id}.pdf.extract.claude.json (lockfile 보호)

PDF 뷰어/워크스페이스에서 파일 우선순위:
  .pdf.extract.claude.json 있으면 사용, 없으면 .pdf.extract.json fallback
  (_best_extract_path() in views.py)
```

### 주요 스크립트

| 스크립트 | 위치 | 역할 |
|---|---|---|
| `ocr_cron.sh` | `/srv/fsis2026/scripts/` | cron 진입점 (OCR + 정규식 추출) |
| `batch_runpod.py` | `/srv/fsis2026/scripts/` | RunPod OCR 일괄 처리 (병렬, health check) |
| `extract_from_ocr.py` | `/srv/fsis2026/scripts/` | OCR JSON → 정규식 추출 → `.pdf.extract.json` |
| `claude_augment.py` | `/srv/fsis2026/scripts/` | regex 결과 → Claude 보강 → `.pdf.extract.claude.json` (lockfile: `/tmp/claude_augment.lock`) |
| `fix_coord_precision.py` | 프로젝트 `scripts/` | 좌표 정밀도 사후 수정 (일회성) |

**스크립트 배포**: `build.sh`에서 Docker 빌드 시 `scripts/*.py`를 `/srv/fsis2026/scripts/`에 자동 복사. 단 `ocr_cron.sh`는 자동 복사 대상이 아니어서 수정 시 수동 복사 필요:

```bash
cp scripts/ocr_cron.sh /srv/fsis2026/scripts/ocr_cron.sh
```

## 공유 볼륨 구조

```
/srv/fsis2026/uploads/references/
├── {year}/
│   ├── {id}.pdf                       — 원본 PDF
│   ├── {id}.pdf.json                  — OCR 결과 (RunPod Chandra2)
│   ├── {id}.pdf.extract.json          — 정규식 추출 결과 (좌표/taxa/표본)
│   └── {id}.pdf.extract.claude.json   — Claude 보강 결과 (우선 사용)
```

- Docker 컨테이너와 호스트가 **동일 디렉토리를 볼륨 마운트로 공유**
- Docker(root)가 PDF 업로드 → 호스트(honestjung)가 OCR/추출 → Docker가 결과 읽기
- **`umask 002`** + `chmod g+w`로 양쪽 모두 읽기/쓰기 가능
  - Docker entrypoint.sh 상단에 `umask 002`
  - 이유: 호스트 cron(honestjung 사용자, devops 그룹)이 같은 디렉토리에 OCR JSON 쓸 수 있게

## 환경변수 파일 (3개)

### `/srv/fsis2026/.env` (Docker Compose)
```
IMAGE_TAG=0.3.16
```

### `/srv/fsis2026/.env.django` (Django)
```
SECRET_KEY=...
DEBUG=False
ALLOWED_HOSTS=fsis.psok.or.kr,34.64.158.160,localhost
DATABASE_PATH=/app/db.sqlite3
MEDIA_ROOT=/app/uploads
SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True
SECURE_HSTS_SECONDS=3600
```

### `/srv/fsis2026/scripts/.env` (RunPod)
```
RUNPOD_API_KEY=...
RUNPOD_ENDPOINT_ID=2vk4pcv7kkn2ax
```

## 향후 고려사항 (server_architecture.md 기준)

- **Docker 내 cron**: RunPod API 키를 `.env.django`에 포함하면 Docker 안에서 OCR 실행 가능. 별도 cron 컨테이너 또는 entrypoint에 cron 데몬 추가 필요.
- **deploy.sh 스크립트 동기화**: 배포 시 `scripts/ocr_cron.sh`를 `/srv/fsis2026/scripts/`에 자동 복사하도록 추가.
- **PostgreSQL 전환**: SQLite WAL 모드의 동시성 한계. 사용자 증가 시 검토.
- **API 기반 추출**: Claude CLI 대신 Anthropic API 직접 사용 시 비용 예측 가능, rate limit 명확.
- **Claude 보강 완료 후**: cron에서 `claude_augment.py` 항목 제거 또는 주석 처리 (불필요한 실행 방지).

## 관련 페이지

- [GHDB](ghdb.md) — GHDB 배포 세부사항
- [PDF 파이프라인](pdf-pipeline.md) — OCR cron 연계, Claude 추출 파일 우선순위
- [KOFHIN 매뉴얼](kofhin-manuals.md) — 관리자 운영 가이드
- [프로젝트 개요](fsis2026-overview.md)

---
*Sources: 002, 010, 011, 015, 016, 017, 021, 022, 023, 024, 025, 026, 027, 028, 052, 060, P10, P11, P12, P13, P16, docs/server_architecture.md*
