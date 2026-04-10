# 009: Django Dockerize 완료

## 작업 내용

P10 계획에 따라 Django 프로젝트를 Docker 컨테이너로 전환.

## 생성 파일

| 파일 | 설명 |
|------|------|
| `Dockerfile` | python:3.12-slim 기반, Pillow 의존성, entrypoint 방식 |
| `entrypoint.sh` | collectstatic + migrate 후 gunicorn 실행 |
| `docker-compose.yml` | 운영용 (`/srv/fsis2026/` 볼륨 마운트) |
| `docker-compose.dev.yml` | 개발용 (로컬 경로 마운트) |
| `.dockerignore` | uploads, staticfiles, DB, map_tiles, .git 등 제외 |

## 주요 결정사항

### collectstatic은 entrypoint에서 실행
- Dockerfile에서 빌드 시 실행하면 이미지 내부에 결과가 들어감
- 그런데 docker-compose에서 staticfiles를 볼륨 마운트하면 호스트 디렉토리가 이미지 내부를 덮어씀
- 따라서 entrypoint.sh에서 컨테이너 시작 시 collectstatic 실행 → 호스트 볼륨에 출력

### migrate도 entrypoint에서 자동 실행
- 컨테이너 재시작/업데이트 시 마이그레이션 자동 적용

## requirements.txt 수정
- `django-appconf`, `django-imagekit`, `pilkit` 누락분 추가

## Docker Hub 배포
- `honestjung/fsis2026:latest`
- `honestjung/fsis2026:0.1.0`

## 테스트 결과
- `docker-compose.dev.yml`로 로컬 테스트 완료
- collectstatic 214 files, migrate 정상, gunicorn worker 3개 기동
- HTTP 302 응답 확인 (로그인 리다이렉트)

## 운영 서버 배포 절차
1. `docker pull honestjung/fsis2026:0.1.0`
2. nginx 설정 변경 (socket → `proxy_pass http://127.0.0.1:8000`)
3. 기존 gunicorn systemd 서비스 중지/비활성화
4. `docker compose up -d`
5. nginx 재시작
