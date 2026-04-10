# P13: DEBUG, ALLOWED_HOSTS를 .env로 분리

## 배경

- `settings.py`에 `DEBUG = True`, `ALLOWED_HOSTS = ['*']`가 하드코딩되어 있음
- 운영 환경에서는 `DEBUG = False`, `ALLOWED_HOSTS`는 실제 IP로 제한해야 함
- Docker 이미지는 하나로 유지하고, 환경별 설정은 `.env`로 분리하는 게 바람직

## 현재 구조

```
Docker 이미지 (honestjung/fsis2026)
  └── /app/              ← 소스코드, settings.py 포함

서버 (/srv/fsis2026/)
  └── .env               ← 볼륨 마운트로 컨테이너에 전달
```

`.env` 파일은 Docker 이미지 밖에 있고, `docker run -v /srv/fsis2026/.env:/app/.env`로 마운트된다.
따라서 이미지를 다시 빌드하지 않아도 `.env`만 수정하면 설정 변경 가능.

## 변경 내용

### settings.py 수정

```python
# 변경 전
DEBUG = True
ALLOWED_HOSTS = ['*']

# 변경 후
DEBUG = config("DEBUG", default=False, cast=bool)
ALLOWED_HOSTS = config("ALLOWED_HOSTS", default="localhost,127.0.0.1", cast=lambda v: [s.strip() for s in v.split(',')])
```

`python-decouple`의 `config()`로 `.env`에서 읽음 (SECRET_KEY와 동일한 방식).

### 운영 서버 .env (`/srv/fsis2026/.env`)

```
SECRET_KEY=...
DEBUG=False
ALLOWED_HOSTS=34.64.158.160,localhost,127.0.0.1
```

### 개발 환경 .env (`~/projects/fsis2026/.env`)

```
SECRET_KEY=...
DEBUG=True
ALLOWED_HOSTS=*
```

## 적용 절차

1. `settings.py` 수정 (위 내용)
2. `/srv/fsis2026/.env`에 `DEBUG`, `ALLOWED_HOSTS` 추가
3. 개발 환경 `.env`에도 `DEBUG=True`, `ALLOWED_HOSTS=*` 추가
4. 다음 Docker 이미지 빌드 시 반영
5. 컨테이너 재시작하면 적용됨
