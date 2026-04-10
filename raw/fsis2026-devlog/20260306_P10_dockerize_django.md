# P10: Django 프로젝트 Dockerize 계획

## 개요

nginx는 호스트에서 그대로 유지하고, fsis2026 Django 프로젝트 전체를 Docker 컨테이너로 전환한다.

## 현재 구성

- **웹서버**: nginx (호스트) → gunicorn unix socket (`/run/gunicorn-fsis.sock`)
- **앱서버**: gunicorn (systemd 서비스 `gunicorn-fsis`)
- **DB**: SQLite3 (`db.sqlite3`)
- **Static**: `collectstatic` → `staticfiles/` (553MB, map_tiles 54,610개 포함)
- **Media**: `uploads/` (8.3GB)
- **설정**: `.env` 파일에서 `python-decouple`로 SECRET_KEY 등 로드

## 디렉토리 구조 분리

현재 `~/projects/fsis2026/` 하나에 소스코드와 운영 데이터가 섞여 있다.
Docker 전환 시 소스코드(빌드용)와 운영 데이터(서비스용)를 분리한다.

### 소스코드 — `~/projects/fsis2026/` (git repo, Docker 빌드용)

```
/home/honestjung/projects/fsis2026/
├── config/
├── fsis/
├── kprdb/
├── evaluator/
├── static/              # 소스 static (map_tiles 제외, .gitignore)
├── Dockerfile
├── docker-compose.yml
└── ...
```

### 운영 데이터 — `/srv/fsis2026/`

```
/srv/fsis2026/
├── db.sqlite3           # DB (또는 향후 PostgreSQL)
├── uploads/             # 미디어 파일 (8.3GB)
├── staticfiles/         # collectstatic 결과 (nginx 서빙)
├── map_tiles/           # 지도 타일 (nginx 직접 서빙)
├── .env                 # 환경변수
└── logs/                # (선택) 로그
```

### 왜 `/srv/`인가

- 리눅스 FHS 표준에서 `/srv`는 "서비스에서 제공하는 데이터"용 디렉토리
- 사용자 홈(`~`)과 분리되어 백업, 권한 관리가 명확
- 백업 시 `/srv/fsis2026/`만 하면 됨

## 고려 사항

### 1. 데이터베이스 (SQLite3)

컨테이너 내부에 DB를 두면 재시작 시 데이터 유실. 선택지:
- **볼륨 마운트** (단순): `-v /srv/fsis2026/db.sqlite3:/app/db.sqlite3`
- **PostgreSQL 전환** (정석): Docker 환경에서 권장. 별도 컨테이너 또는 호스트 DB로 분리

### 2. 미디어 파일 (8.3GB)

`uploads/` 디렉토리를 반드시 볼륨 마운트로 분리:
```
-v /srv/fsis2026/uploads:/app/uploads
```

### 3. Static 파일 및 Map Tiles

#### Static 파일
collectstatic 결과를 호스트 경로에 써야 nginx가 서빙 가능:
```
-v /srv/fsis2026/staticfiles:/app/staticfiles
```

#### Map Tiles 분리
map_tiles(54,610개 PNG, ~553MB 중 대부분)는 Django/Docker와 무관하게 서버에 고정 배치.
nginx에서 별도 location으로 직접 서빙:

```nginx
# 맵타일 — 별도 경로로 직접 서빙 (긴 경로가 우선 매칭)
location /static/fsis/map_tiles/ {
    alias /srv/fsis2026/map_tiles/;
}

# 나머지 static
location /static/ {
    alias /srv/fsis2026/staticfiles/;
}
```

클라이언트 URL(`/static/fsis/map_tiles/...`)은 변경 없이 유지되므로 Django 템플릿 수정 불필요.

### 4. 환경변수 (.env)

`python-decouple`은 환경변수도 읽으므로 두 가지 방법 가능:
- `.env` 파일 마운트: `-v /srv/fsis2026/.env:/app/.env`
- Docker 환경변수 전달: `docker run -e SECRET_KEY=xxx`

### 5. nginx 설정 변경

unix socket → TCP 포트로 변경 필요:

```nginx
# 변경 전
proxy_pass http://unix:/run/gunicorn-fsis.sock;

# 변경 후
proxy_pass http://127.0.0.1:8000;
```

### 6. 기존 gunicorn systemd 서비스

Docker가 gunicorn을 실행하므로 기존 서비스 비활성화:
```bash
sudo systemctl stop gunicorn-fsis
sudo systemctl disable gunicorn-fsis.socket gunicorn-fsis.service
```

### 7. .dockerignore

Docker 이미지 경량화를 위해 대용량/불필요 파일 제외:
```
staticfiles/
uploads/
db.sqlite3
nkfsite/
*.sqlite3
static/fsis/map_tiles/
devlog/
docs/
.git/
__pycache__/
*.pyc
.env
```

## Dockerfile 초안

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "3", "config.wsgi:application"]
```

## docker-compose.yml 초안

```yaml
services:
  django:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - /srv/fsis2026/db.sqlite3:/app/db.sqlite3
      - /srv/fsis2026/uploads:/app/uploads
      - /srv/fsis2026/staticfiles:/app/staticfiles
    env_file:
      - /srv/fsis2026/.env
    restart: unless-stopped
```

## nginx 최종 설정 (안)

```nginx
server {
    listen 80;
    server_name _;

    location /static/fsis/map_tiles/ {
        alias /srv/fsis2026/map_tiles/;
    }

    location /static/ {
        alias /srv/fsis2026/staticfiles/;
    }

    location /media/ {
        alias /srv/fsis2026/uploads/;
    }

    location / {
        include proxy_params;
        proxy_pass http://127.0.0.1:8000;
    }
}
```

## 작업 이력

### 2026-03-06: `/srv/fsis2026/` 초기 세팅 완료

gunicorn-fsis 서비스 중지 후, `~/projects/fsis2026/`에서 운영 데이터를 복사함.
원본은 아직 그대로 유지 중 (Docker 전환 확인 후 정리 예정).

```bash
sudo mkdir -p /srv/fsis2026/{uploads,staticfiles,map_tiles,logs}
sudo cp    ~/projects/fsis2026/db.sqlite3  /srv/fsis2026/
sudo cp    ~/projects/fsis2026/.env        /srv/fsis2026/
sudo cp -a ~/projects/fsis2026/uploads/.   /srv/fsis2026/uploads/
sudo cp -a ~/projects/fsis2026/staticfiles/. /srv/fsis2026/staticfiles/
sudo cp -a ~/projects/fsis2026/staticfiles/fsis/map_tiles/. /srv/fsis2026/map_tiles/
sudo chown -R honestjung:www-data /srv/fsis2026/
```

결과:
| 항목 | 크기 |
|------|------|
| db.sqlite3 | 9.8MB |
| uploads/ | 8.3GB |
| staticfiles/ | 553MB |
| map_tiles/ | 550MB |
| .env | 62B |
| logs/ | (빈 디렉토리) |

현재 서비스는 기존 경로(`~/projects/fsis2026/`)에서 gunicorn으로 계속 운영 중.

## 배포 절차 (안)

1. ~~운영 데이터 이동~~ → 완료 (위 작업 이력 참조)
2. Docker 이미지 빌드 (개발 서버 또는 CI)
3. 이미지를 운영 서버로 전달 (registry 또는 `docker save/load`)
4. 기존 gunicorn 서비스 중지/비활성화
5. nginx 설정 변경 (socket → TCP, 경로를 `/srv/fsis2026/`로)
6. `docker compose up -d`
7. nginx 재시작

## 향후 검토

- SQLite → PostgreSQL 전환 여부
- HTTPS/SSL 설정 (certbot 등)
- Docker 이미지 CI/CD 파이프라인
- 컨테이너 로그 관리
- `DEBUG = True` → 운영 환경 설정 분리
