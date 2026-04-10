# P16: 도메인 연결 및 프로덕션 보안 설정

**작성일**: 2026-03-14
**상태**: 진행 중

## 배경

테스트용 도메인(`geolmapkr.nopeoplestime.info`)에서 정식 도메인 `fsis.psok.or.kr`을 확보하여
Let's Encrypt 인증서 적용 완료. 이에 따라 Django 프로덕션 보안 설정 활성화 필요.

## 완료 항목

- [x] 정식 도메인 확보: `fsis.psok.or.kr`
- [x] Let's Encrypt SSL 인증서 발급/적용

## 작업 항목

### 1. ALLOWED_HOSTS 업데이트

양쪽 `.env` 파일에 새 도메인 추가:
- `/srv/fsis2026/.env`
- `/srv/ghdb/.env`

```
ALLOWED_HOSTS=fsis.psok.or.kr,localhost,127.0.0.1
```

### 2. Nginx 설정 업데이트

- `server_name`을 `fsis.psok.or.kr`로 변경
- `X-Forwarded-Proto` 헤더 전달 확인:
  ```nginx
  proxy_set_header X-Forwarded-Proto $scheme;
  ```
- 이 헤더가 있어야 Django의 `SECURE_PROXY_SSL_HEADER` 설정이 정상 동작

### 3. Django 보안 설정 활성화 (.env)

`config/settings.py`에 이미 `.env` 제어로 준비됨 (128~136행). 아래 값을 `.env`에 추가:

| 설정 | 값 | 설명 |
|------|----|------|
| `SESSION_COOKIE_SECURE` | `True` | 세션 쿠키 HTTPS 전용 |
| `CSRF_COOKIE_SECURE` | `True` | CSRF 쿠키 HTTPS 전용 |
| `SECURE_HSTS_SECONDS` | `3600` | HSTS 1시간 (테스트) |
| `SECURE_HSTS_INCLUDE_SUBDOMAINS` | `True` | 서브도메인 HSTS |
| `SECURE_HSTS_PRELOAD` | `False` | preload는 안정화 후 |
| `SECURE_SSL_REDIRECT` | `False` | Nginx에서 처리 (아래 참고) |

### 4. SSL 리다이렉트 주의사항

`SECURE_SSL_REDIRECT`는 Nginx에서 이미 HTTP→HTTPS 리다이렉트를 하고 있으면
Django에서 중복 리다이렉트가 발생하므로 `False`로 유지.
Nginx 설정에서 리다이렉트를 처리하는 것이 표준 구성.

### 5. HSTS 단계적 적용

HSTS는 한번 브라우저에 캐시되면 `max-age` 동안 되돌리기 어려움:
1. **1단계**: `SECURE_HSTS_SECONDS=3600` (1시간) — 문제 없는지 확인
2. **2단계**: `SECURE_HSTS_SECONDS=86400` (1일)
3. **3단계**: `SECURE_HSTS_SECONDS=31536000` (1년) + `SECURE_HSTS_PRELOAD=True`

### 6. 적용 후 검증

```bash
# Django 보안 체크
python manage.py check --deploy

# HTTPS 응답 헤더 확인
curl -I https://fsis.psok.or.kr/

# HSTS 헤더 확인
curl -sI https://fsis.psok.or.kr/ | grep -i strict

# HTTP→HTTPS 리다이렉트 확인
curl -I http://fsis.psok.or.kr/
```

## 적용 대상

fsis / ghdb 양쪽 `.env` 모두 동일하게 적용 필요:
- `/srv/fsis2026/.env` (fsis 컨테이너)
- `/srv/ghdb/.env` (ghdb 컨테이너)

설정 변경 후 컨테이너 재시작:
```bash
docker restart fsis ghdb
```
