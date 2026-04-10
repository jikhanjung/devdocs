# 015: 표본 사진 라이트박스 + deploy 디렉토리 정리

## 작업 내용

### 1. 표본정보 상세 페이지 사진 라이트박스

`fsis/templates/fsis/museum_specimen_detail.html`

- 기존: 썸네일 클릭 → 새 창(`target="_blank"`)으로 원본 사진 열기
- 변경: 썸네일 클릭 → 같은 페이지에서 라이트박스 오버레이로 원본 사진 표시
- 닫기: 배경 클릭, × 버튼, Esc 키
- 외부 라이브러리 없이 순수 JS/CSS 구현
- prev/next 기능: 좌우 화살표 버튼 + 키보드 방향키(←→) 지원, 순환 이동
- 사진 1장이면 화살표 숨김

### 2. Docker 파일 deploy/ 디렉토리로 이동

프로젝트 루트의 Docker 관련 파일을 `deploy/`로 정리:

| 파일 | 이동 |
|------|------|
| `Dockerfile` | `deploy/Dockerfile` |
| `entrypoint.sh` | `deploy/entrypoint.sh` |
| `docker-compose.yml` | `deploy/docker-compose.yml` |
| `docker-compose.dev.yml` | `deploy/docker-compose.dev.yml` |

추가 파일:
- `deploy/DOCKER_VERSION` — Docker 이미지 버전 기록 (현재 0.1.7)

변경 사항:
- `Dockerfile`: entrypoint 경로를 `deploy/entrypoint.sh`로 변경
- `docker-compose*.yml`: build context를 `..`(프로젝트 루트), dockerfile을 `deploy/Dockerfile`로 지정
- `.dockerignore`는 빌드 컨텍스트(프로젝트 루트)에 있어야 하므로 그대로 유지

### 빌드/배포 명령어 (프로젝트 루트에서)

```bash
VER=$(cat deploy/DOCKER_VERSION)
docker build -f deploy/Dockerfile -t honestjung/fsis2026:$VER -t honestjung/fsis2026:latest .
docker push honestjung/fsis2026:$VER && docker push honestjung/fsis2026:latest
```
