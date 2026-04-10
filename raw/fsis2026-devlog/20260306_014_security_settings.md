# 014: 프로덕션 보안 설정 준비

## 작업 내용

### 1. Django 보안 설정 추가 (`config/settings.py`)

HTTPS 적용 전까지 안전하게 `.env`로 제어 가능하도록 보안 설정 추가.
기본값은 모두 비활성(False/0)이므로 현재 HTTP 운영에 영향 없음.

추가된 설정:
- `SECURE_SSL_REDIRECT` — HTTP→HTTPS 리다이렉트
- `SESSION_COOKIE_SECURE` — 세션 쿠키 HTTPS 전용
- `CSRF_COOKIE_SECURE` — CSRF 쿠키 HTTPS 전용
- `SECURE_HSTS_SECONDS` — HSTS 헤더 (기본 0)
- `SECURE_HSTS_INCLUDE_SUBDOMAINS` — 서브도메인 포함
- `SECURE_HSTS_PRELOAD` — HSTS preload 등록
- `SECURE_PROXY_SSL_HEADER` — Nginx 프록시 뒤에서 HTTPS 감지 (`DEBUG=False`일 때만)

### 2. HANDOFF.md 갱신

- P07~P09, P10, P12, P13 완료 상태 반영 (HANDOFF.md에 미반영되어 있던 항목)
- Docker 전환에 맞게 서비스 관리 명령어 갱신
- 프로젝트 구조에 evaluator/, Docker 파일 추가
- 카카오맵 관련 내용 제거, MapLibre로 갱신
- Next Steps를 실제 미완료 3건으로 정리

## 보안 점검 방법

서버에서 Django 프로덕션 보안 점검 실행:

```bash
docker exec fsis2026 python manage.py check --deploy
```

이 명령은 `settings.py`를 검사하여 프로덕션에서 놓치기 쉬운 보안 설정을 경고로 알려준다.
컨테이너 내부에서 서버의 `.env` 설정(`DEBUG=False` 등)을 읽고 점검하므로
실제 운영 환경 기준으로 진단된다.

현재 상태에서는 HTTPS 관련 경고(W004, W008, W012, W016 등)가 나온다.
도메인+HTTPS 적용 후 `.env`에서 보안 설정을 켜고 재점검하면 경고가 해소된다.

## 도메인 + HTTPS 적용 후 할 일

1. Let's Encrypt 인증서 발급
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d your-domain.com
   ```

2. `.env`에 보안 설정 추가
   ```
   SECURE_SSL_REDIRECT=True
   SESSION_COOKIE_SECURE=True
   CSRF_COOKIE_SECURE=True
   SECURE_HSTS_SECONDS=31536000
   SECURE_HSTS_INCLUDE_SUBDOMAINS=True
   SECURE_HSTS_PRELOAD=True
   ```

3. 컨테이너 재시작 후 재점검
   ```bash
   docker restart fsis2026
   docker exec fsis2026 python manage.py check --deploy
   ```
