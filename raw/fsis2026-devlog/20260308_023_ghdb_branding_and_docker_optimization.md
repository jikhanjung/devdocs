# 023: GHDB 브랜딩 변경 및 Docker 이미지 최적화

**날짜**: 2026-03-08

## 작업 내용

### 1. GHDB 브랜딩 변경

- navbar 브랜드: `GHDB` → `2026 지질유산 데이터베이스` (`ghdb/templates/ghdb/navbar.html`)
- 브라우저 탭 제목: `지질유산 표본 DB` → `2026 지질유산 데이터베이스` (`ghdb/templates/ghdb/base.html`)

### 2. Docker 이미지에서 맵타일 제외

`.dockerignore`의 맵타일 경로가 잘못되어 550MB 맵타일이 Docker 이미지에 포함되고 있었음:
- 수정 전: `static/fsis/map_tiles/` (매칭 안 됨)
- 수정 후: `fsis/static/fsis/map_tiles/` (정상 제외)

효과:
- Docker 이미지 크기 ~550MB 감소
- `collectstatic` 시 54,000+개 파일 처리 제거 → 컨테이너 시작 속도 개선
- 맵타일은 nginx에서 직접 서비스 (`/srv/fsis2026/map_tiles/`)

### 3. Docker 버전 0.2.2

변경 사항 반영하여 DOCKER_VERSION 0.2.1 → 0.2.2 업데이트.

### 4. 디스크 정리

빌드 중 `no space left on device` 오류 발생 → `docker system prune -a`로 미사용 이미지/캐시 정리 (약 7GB 회수, 48GB 중 9.4GB 여유 확보).
