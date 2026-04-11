# Docker 배포 (프로덕션 웹 뷰어)

scoda-engine의 **프로덕션 Docker 이미지**는 읽기 전용 웹 뷰어를 단일 컨테이너로 배포한다. 2026-03-02에 첫 구현(P22) 후 2-컨테이너 → 1-컨테이너 → nginx 제거까지 단순화되어, 현재는 gunicorn이 단일 포트(8081)에서 직접 서빙한다.

> FSIS2026의 [배포 인프라](deployment.md)와는 별개 — 이 페이지는 scoda-engine server 이미지를 다룬다.

## 진화

```
v0.1.0 (03-02)  — 2-컨테이너 (nginx + gunicorn), Flask
     ↓
v0.1.0 (03-02)  — 단일 컨테이너 (nginx + gunicorn 동일 이미지)
     ↓
v0.1.0 (03-06)  — nginx 제거, gunicorn 직접 서빙 포트 8081
     ↓
v0.1.0+ (03-14) — Multi-Package Serving + Vendor JS 내장 → 완전 오프라인 가능
```

## 주요 컴포넌트

### `scoda_engine/serve_web.py` (03-02, 029)

gunicorn 앱 팩토리. `serve.py` (Desktop CLI)와 달리 production 지향:

- **MCP opt-in**: 기본적으로 MCP 서버 비활성화 (프로덕션 노출 방지)
- **Viewer 모드 기본**: CRUD 쓰기 API 비노출
- `/healthz` 엔드포인트 (Docker healthcheck, k8s readiness probe)
- CLI: `python -m scoda_engine.serve_web --port 8081 --packages-dir /data`

### `deploy/Dockerfile`

```dockerfile
# 단계 1: 빌드 타임 Hub 패키지 다운로드
RUN python deploy/fetch_packages.py

# 단계 2: 런타임 gunicorn
CMD gunicorn "scoda_engine.serve_web:create_app()" \
    --bind 0.0.0.0:8081 --workers 2
```

### `deploy/fetch_packages.py` (03-02, 030)

Docker 빌드 시 Hub에서 최신 `.scoda`를 자동 다운로드:
- `hub_client.py` 사용
- `SCODA_HUB_SSL_VERIFY=0` 환경변수로 기관 네트워크 SSL 우회 (03-06, 035)
- 다운로드된 패키지는 이미지 내 `/data`에 고정 → 런타임 외부 접속 불필요
- 상세: [SCODA Hub](scoda-hub.md)

### `deploy/docker-compose.yml`

- 서비스 하나 (`scoda-server`)
- 포트 매핑 `8081:8081`
- 환경변수 `SCODA_ENGINE_NAME` (아래 참조)
- `bump_version.py`로 버전 자동 갱신 (03-14, 032)

## 컨테이너 단순화 여정

### 2-컨테이너 → 1-컨테이너 (03-02, 031)

- 초기 (029): `nginx` 컨테이너(정적/프록시) + `app` 컨테이너(gunicorn) 분리
- 통합 (031): 같은 이미지에서 nginx + gunicorn 동시 실행 (supervisord)
- 장점: 외부 네트워크 설정 단일화

### nginx 완전 제거 (03-06, 034)

- **이유**: gunicorn이 이미 정적 파일 서빙 가능, nginx 레이어가 불필요한 복잡도
- gunicorn을 포트 8081에서 직접 바인딩
- Dockerfile에서 nginx 패키지 제거 → 이미지 크기 축소
- supervisord도 제거 → 단일 프로세스 컨테이너 (k8s 친화적)

## SCODA_ENGINE_NAME 환경변수 (03-06, 035)

동일 코드베이스로 Desktop과 Server를 구분 표시하기 위한 환경변수:

| 환경 | `SCODA_ENGINE_NAME` | Navbar 표시 |
|---|---|---|
| Desktop EXE | (기본값) `SCODA Desktop` | "Powered by SCODA Desktop v0.3.x" |
| Docker 프로덕션 | `SCODA Server` | "Powered by SCODA Server v0.3.x" |

버전 표시(018)와 결합하여 런타임 식별 가능.

## 릴리스 자동화 (03-06, 037)

`.github/workflows/release.yml`에 Docker Hub 빌드·push 추가:

1. PyInstaller EXE 빌드 (기존)
2. **Docker 이미지 빌드** (`deploy/Dockerfile`)
3. Docker Hub `honestjung/scoda-server:{version}` + `:latest` 태그 push
4. `SCODA_HUB_SSL_VERIFY=0`로 빌드 타임 SSL 우회

상세: [릴리스 워크플로우](scoda-engine-release.md)

## bump_version.py 스크립트 (03-06/03-14, 038/032)

버전 불일치 방지:
- `pyproject.toml` version
- `scoda_engine/__init__.py` `__version__`
- `deploy/docker-compose.yml` image tag (03-14 추가)

CLI: `python scripts/bump_version.py 0.3.2`

## GCP 배포 검증

- 2026-03-02: 최초 GCP 서버에 배포 성공
- Docker Hub `honestjung/scoda-server:0.1.0` 게시 확인
- trilobase 패키지 자동 포함 (build-time fetch)

## 배포 후 기능 추가

- **Hub Refresh API** (03-11, 043 / 03-12, P27): 서버 재시작 없이 Hub 패키지 갱신 가능. Navbar `↻` 버튼으로 트리거.
- **Multi-Package Serving** (v0.3.0, 03-14, P29): 단일 인스턴스에서 여러 `.scoda` 동시 서빙. 상세: [multi-package-serving.md](multi-package-serving.md).
- **Vendor JS 내장** (03-14, 031): D3/Bootstrap 로컬 번들 → CDN 차단 환경 대응.
- **Meta-Package 지원** (03-18): paleobase 합성 뷰. [meta-package.md](meta-package.md).

## 관련 페이지

- [scoda-engine](scoda-engine.md) — 런타임 본체
- [SCODA Hub](scoda-hub.md) — `fetch_packages.py` 빌드 타임 사용
- [Multi-Package Serving](multi-package-serving.md) — `/api/{package}` 라우팅
- [CRUD 프레임워크](crud-framework.md) — Viewer 모드 제약
- [릴리스 워크플로우](scoda-engine-release.md) — Docker Hub CI
- FSIS2026 [배포 인프라](deployment.md) — 별개의 배포 환경 (교차 참고)

---
*Sources: P22, 029-031, 034-035, 037, 038, 032(03-14), 043, P27.*
