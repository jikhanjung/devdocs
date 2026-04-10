# P12: Docker 배포 실행 기록

## 개요

P10 계획에 따라 fsis2026 Django 프로젝트를 Docker 컨테이너로 전환 완료.

## 작업 내용 (2026-03-06)

### 1. 기존 서비스 중지

```bash
sudo systemctl stop gunicorn-fsis.socket gunicorn-fsis.service
```

### 2. Docker 이미지 pull 및 실행

```bash
docker pull honestjung/fsis2026
docker run -d \
  --name fsis2026 \
  -p 8000:8000 \
  -v /srv/fsis2026/db.sqlite3:/app/db.sqlite3 \
  -v /srv/fsis2026/uploads:/app/uploads \
  -v /srv/fsis2026/staticfiles:/app/staticfiles \
  -v /srv/fsis2026/.env:/app/.env \
  --restart unless-stopped \
  honestjung/fsis2026
```

### 3. nginx 설정 변경

`/etc/nginx/sites-enabled/fsis2026`:
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

변경 사항:
- `proxy_pass`: unix socket → `http://127.0.0.1:8000`
- static/media 경로: `~/projects/fsis2026/` → `/srv/fsis2026/`
- map_tiles: 별도 location으로 분리 (nginx 직접 서빙)
- 기존 설정 백업: `/etc/nginx/fsis2026.bak`

### 4. 동작 확인

- `/fsis/` → 302 (로그인 리다이렉트, 정상)
- `/fsis/museum_specimen_list/` → 200 (정상)
- nginx access log로 확인 완료

## 컨테이너 entrypoint

```bash
#!/bin/bash
set -e
python manage.py collectstatic --noinput
python manage.py migrate --noinput
exec gunicorn --bind 0.0.0.0:8000 --workers 3 config.wsgi:application
```

컨테이너 시작 시 자동으로 collectstatic과 migrate 실행.

## 로그 확인

```bash
# 컨테이너 로그 (gunicorn 시작/에러)
docker logs fsis2026

# nginx access log
sudo tail -f /var/log/nginx/access.log
```

## 알려진 이슈

- **gunicorn access log 미출력**: entrypoint.sh에서 `--access-logfile -` 옵션이 빠져있어 `docker logs`에 요청별 access log가 나오지 않음. 다음 이미지 빌드 시 추가 필요:
  ```bash
  exec gunicorn --bind 0.0.0.0:8000 --workers 3 --access-logfile - config.wsgi:application
  ```

## 현재 상태

- gunicorn-fsis systemd 서비스: 중지됨 (disabled 상태)
- Docker 컨테이너 `fsis2026`: 실행 중 (`--restart unless-stopped`)
- nginx: `/srv/fsis2026/` 경로로 서빙, TCP 8000 프록시
- `~/projects/fsis2026/`의 운영 데이터 원본: 아직 유지 중 (안정 확인 후 정리 예정)
