# 서버 구성 및 운영 아키텍처

**최종 수정**: 2026-04-07

---

## 1. 전체 구조

```
프로덕션 서버 (34.64.158.160, GCP)
│
├── Nginx (리버스 프록시, SSL)
│   ├── :443 (HTTPS) → fsis 컨테이너 :8000
│   └── :8001 (SSL)  → ghdb 컨테이너 :8002
│
├── Docker 컨테이너
│   ├── fsis (config.settings_fsis) — 메인 앱
│   └── ghdb (config.settings_ghdb) — 지질유산 DB
│
├── 호스트 서비스
│   ├── cron (OCR + 정규식 추출 + Claude 보강, 10분 간격)
│   └── 서버 전용 venv (/srv/fsis2026/venv/)
│
└── 공유 볼륨
    ├── /srv/fsis2026/db.sqlite3
    ├── /srv/fsis2026/uploads/references/ (PDF, OCR JSON, 추출 JSON)
    └── /srv/fsis2026/staticfiles/
```

---

## 2. Docker 컨테이너

### 역할
- Django 웹 서빙 (Gunicorn)
- 정적 파일 수집 (`collectstatic`)
- DB 마이그레이션 (`migrate`)

### 환경 설정
- `.env` — Docker Compose용 (이미지 태그, 포트 등)
- `.env.django` — Django 환경변수 (SECRET_KEY, DATABASE_PATH, MEDIA_ROOT, 보안 설정 등)

### 실행
```bash
# entrypoint.sh
umask 002                          # group 쓰기 권한 보장
python manage.py collectstatic --noinput
python manage.py migrate --noinput
exec gunicorn --bind 0.0.0.0:8000 --workers ${GUNICORN_WORKERS:-2} ...
```

### umask 002
Docker 컨테이너(root)가 생성하는 파일/디렉토리에 group 쓰기 권한을 부여.
호스트의 cron(honestjung 사용자, devops 그룹)이 같은 디렉토리에 OCR JSON을 쓸 수 있도록 함.

---

## 3. 호스트 OCR/추출 파이프라인

### 왜 Docker 밖에서 실행하는가
- **RunPod API 키**: `/srv/fsis2026/scripts/.env`에만 존재, Docker 이미지에 포함하지 않음
- **별도 venv**: `/srv/fsis2026/venv/` — OCR용 최소 패키지 (PyMuPDF, requests, python-dotenv)
- **독립 실행**: Django 의존 없이 스크립트 단독 실행 가능
- Docker 컨테이너는 웹 서빙에만 집중

### 서버 전용 venv
```
/srv/fsis2026/venv/
├── PyMuPDF       — PDF 페이지 렌더링 (이미지 → RunPod 전송)
├── requests      — RunPod API 호출
├── python-dotenv — .env 파일 읽기
└── Pillow        — 이미지 처리
```

### cron 구성
```
# OCR (증분, .last_ocr_run 기반)
*/10 * * * * /srv/fsis2026/scripts/ocr_cron.sh >> /srv/fsis2026/scripts/ocr_cron.log 2>&1

# Claude 보강 (1개씩, 오래된 것부터)
*/10 * * * * /srv/fsis2026/venv/bin/python /srv/fsis2026/scripts/claude_augment.py --limit 1 >> /srv/fsis2026/scripts/claude_augment.log 2>&1
```

### 파이프라인 흐름
```
1. cron 실행 (10분 간격)
   ├── OCR: find → batch_runpod.py → {id}.pdf.json
   ├── 정규식 추출: extract_from_ocr.py → {id}.pdf.extract.json
   └── Claude 보강: claude_augment.py → {id}.pdf.extract.claude.json (lockfile 보호)

PDF 뷰어/워크스페이스에서 파일 우선순위:
  .pdf.extract.claude.json 있으면 사용, 없으면 .pdf.extract.json fallback
  (_best_extract_path() in views.py)
```

### 주요 스크립트
| 스크립트 | 위치 | 역할 |
|----------|------|------|
| `ocr_cron.sh` | `/srv/fsis2026/scripts/` | cron 진입점 (OCR + 정규식 추출) |
| `batch_runpod.py` | `/srv/fsis2026/scripts/` | RunPod OCR 일괄 처리 (병렬, health check) |
| `extract_from_ocr.py` | `/srv/fsis2026/scripts/` | OCR JSON → 정규식 추출 → `.pdf.extract.json` |
| `claude_augment.py` | `/srv/fsis2026/scripts/` | regex 결과 → Claude 보강 → `.pdf.extract.claude.json` (lockfile: `/tmp/claude_augment.lock`) |
| `fix_coord_precision.py` | 프로젝트 `scripts/` | 좌표 정밀도 사후 수정 (일회성) |

### 스크립트 배포
`build.sh`에서 Docker 빌드 시 `scripts/*.py`를 `/srv/fsis2026/scripts/`에 자동 복사.
`ocr_cron.sh`는 자동 복사 대상이 아니므로 수정 시 수동 복사 필요:
```bash
cp scripts/ocr_cron.sh /srv/fsis2026/scripts/ocr_cron.sh
```

---

## 4. 공유 볼륨 구조

```
/srv/fsis2026/uploads/references/
├── {year}/
│   ├── {id}.pdf                       — 원본 PDF
│   ├── {id}.pdf.json                  — OCR 결과 (RunPod Chandra2)
│   ├── {id}.pdf.extract.json          — 정규식 추출 결과 (좌표/taxa/표본)
│   └── {id}.pdf.extract.claude.json   — Claude 보강 결과 (우선 사용)
```

- Docker 컨테이너와 호스트가 동일 디렉토리를 볼륨 마운트로 공유
- Docker(root)가 PDF 업로드 → 호스트(honestjung)가 OCR/추출 → Docker가 결과 읽기
- `umask 002` + `chmod g+w`로 양쪽 모두 읽기/쓰기 가능

---

## 5. 환경변수 파일

### /srv/fsis2026/.env (Docker Compose)
```
IMAGE_TAG=0.3.16
```

### /srv/fsis2026/.env.django (Django)
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

### /srv/fsis2026/scripts/.env (RunPod)
```
RUNPOD_API_KEY=...
RUNPOD_ENDPOINT_ID=2vk4pcv7kkn2ax
```

---

## 6. 향후 고려사항

- **Docker 내 cron**: RunPod API 키를 `.env.django`에 포함하면 Docker 안에서 OCR 실행 가능. 별도 cron 컨테이너 또는 entrypoint에 cron 데몬 추가 필요.
- **deploy.sh 스크립트 동기화**: 배포 시 `scripts/ocr_cron.sh`를 `/srv/fsis2026/scripts/`에 자동 복사하도록 추가.
- **PostgreSQL 전환**: SQLite WAL 모드의 동시성 한계. 사용자 증가 시 검토.
- **API 기반 추출**: Claude CLI 대신 Anthropic API 직접 사용 시 비용 예측 가능, rate limit 명확.
- **Claude 보강 완료 후**: cron에서 `claude_augment.py` 항목 제거 또는 주석 처리 (불필요한 실행 방지).
