# 022: HTTPS 설정 및 Nginx 정적 파일 서비스

**날짜**: 2026-03-08

## 배경

- 기존 Apache에서 서비스하던 정적 페이지들(`/var/www/html/`)이 Nginx 전환 후 접근 불가 상태
- 도메인 `geolmapkr.nopeoplestime.info`가 서버 IP(34.64.158.160)에 연결되어 있어 HTTPS 테스트 가능
- HANDOFF.md의 Next Steps에 도메인 연결 및 HTTPS 설정이 남아있었음

## 작업 내용

### 1. Nginx에서 /var/www/html 정적 파일 서비스

이전 Apache document root(`/var/www/html/`)에 다음 콘텐츠가 존재:
- `index.html` — 한국 지질도 페이지로 리다이렉트
- `3D-model/`, `GeologicalMapOfKorea/`, `NKFossil/`, `NKFossilExplorer/`, `pofois/`
- `nkfadmin` → 심볼릭 링크 (원본 프로젝트 정적파일)
- `marinecroc.html`

fsis2026 Nginx 설정에 `root /var/www/html`과 `try_files` 추가:

```nginx
root /var/www/html;
charset utf-8;

location / {
    try_files $uri $uri/ @django;
}

location @django {
    include proxy_params;
    proxy_pass http://127.0.0.1:8000;
}
```

- 정적 파일이 존재하면 직접 서비스, 없으면 Django로 폴백
- `charset utf-8` 추가 (index.html에 meta charset 누락으로 한글 깨짐 해결)

### 2. HTTPS 설정 (Let's Encrypt + Certbot)

#### certbot nginx 플러그인 설치
기존에 `python3-certbot-apache`만 설치되어 있어서 nginx 플러그인 추가:
```bash
sudo apt-get install -y python3-certbot-nginx
```

#### fsis (포트 443)
```bash
sudo certbot --nginx -d geolmapkr.nopeoplestime.info \
  --non-interactive --agree-tos --register-unsafely-without-email --redirect
```
- Certbot이 자동으로 SSL 설정 및 HTTP→HTTPS 리다이렉트 추가
- 인증서 만료: 2026-06-06 (자동 갱신 설정됨)

#### ghdb (포트 8001)
Certbot이 비표준 포트(8001)는 자동 처리하지 못해 수동으로 SSL 설정:
```nginx
server {
    listen 8001 ssl;
    server_name geolmapkr.nopeoplestime.info;

    ssl_certificate /etc/letsencrypt/live/geolmapkr.nopeoplestime.info/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/geolmapkr.nopeoplestime.info/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    ...
}
```

### 3. Django ALLOWED_HOSTS 업데이트

`/srv/fsis2026/.env`와 `/srv/ghdb/.env`에 도메인 추가:
```
ALLOWED_HOSTS=34.64.158.160,localhost,127.0.0.1,geolmapkr.nopeoplestime.info
```

#### 주의: docker restart vs docker compose up -d
- `docker restart`는 env_file을 다시 읽지 않음 (환경변수 변경 미반영)
- `docker compose up -d`로 컨테이너를 재생성해야 .env 변경이 반영됨

```bash
cd deploy && docker compose up -d
```

## 최종 Nginx 설정

### /etc/nginx/sites-available/fsis2026
```nginx
server {
    server_name geolmapkr.nopeoplestime.info;

    root /var/www/html;
    charset utf-8;

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
        try_files $uri $uri/ @django;
    }

    location @django {
        include proxy_params;
        proxy_pass http://127.0.0.1:8000;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/geolmapkr.nopeoplestime.info/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/geolmapkr.nopeoplestime.info/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = geolmapkr.nopeoplestime.info) {
        return 301 https://$host$request_uri;
    }

    listen 80;
    server_name geolmapkr.nopeoplestime.info;
    return 404;
}
```

### /etc/nginx/sites-available/ghdb
```nginx
server {
    listen 8001 ssl;
    server_name geolmapkr.nopeoplestime.info;

    ssl_certificate /etc/letsencrypt/live/geolmapkr.nopeoplestime.info/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/geolmapkr.nopeoplestime.info/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location /static/ {
        alias /srv/ghdb/staticfiles/;
    }

    location /media/ {
        alias /srv/ghdb/uploads/;
    }

    location / {
        include proxy_params;
        proxy_pass http://127.0.0.1:8002;
    }
}
```

## 접속 정보

| 서비스 | URL |
|--------|-----|
| fsis | https://geolmapkr.nopeoplestime.info |
| ghdb | https://geolmapkr.nopeoplestime.info:8001 |
| 정적 페이지 (구 Apache) | https://geolmapkr.nopeoplestime.info/GeologicalMapOfKorea/ 등 |
