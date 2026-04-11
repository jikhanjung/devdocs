# 릴리스 워크플로우 (CI/CD, PyInstaller, MkDocs)

scoda-engine 독립 레포의 릴리스 인프라. 2026-02-23(S-7)에 GitHub Actions CI/CD를 세팅한 후, 02-26 문서 사이트, 03-06 Docker Hub 자동화까지 확장되었다.

## GitHub Actions 워크플로우 (4종)

| 파일 | 트리거 | 용도 |
|---|---|---|
| `.github/workflows/test.yml` | push / PR | 2 OS × 2 Python 매트릭스 테스트 |
| `.github/workflows/release.yml` | tag push (`core-v*`, `desktop-v*`) / manual | PyInstaller + GitHub Release + Docker Hub |
| `.github/workflows/hub-index.yml` | weekly cron + manual | Hub index 생성 + gh-pages |
| `.github/workflows/pages.yml` | docs 변경 / manual | MkDocs + Hub index 통합 gh-pages |

## test.yml (S-7, 02-23, 009)

- **매트릭스**: Ubuntu × Windows, Python 3.10 × 3.12
- 단계: checkout → setup-python → `pip install -e ".[dev]"` → `pytest`
- monorepo `core/` 서브패키지도 함께 테스트
- Badge를 README에 표시

## release.yml (S-7, 02-23, 010 + 02-24, 037)

### 초기 버전 (02-23)

- `workflow_dispatch`(manual trigger) — 안전한 수동 릴리스
- 태그 생성 + `python scripts/release.py` 호출
- PyInstaller 단일 EXE 빌드 (`ScodaDesktop.spec`)
- GitHub Release 생성 + EXE 자산 업로드

### Docker Hub 통합 (03-06, 037)

- PyInstaller EXE 빌드 후 **Docker 이미지 빌드/push 단계** 추가
- `honestjung/scoda-server:{version}` + `:latest`
- `SCODA_HUB_SSL_VERIFY=0` 환경변수로 빌드 타임 Hub 패키지 SSL 우회

상세: [Docker 배포](docker-deployment.md)

### 태그 prefix 분리 (P07)

- `core-v*` → `scoda-engine-core` 빌드 (PyPI twine upload)
- `desktop-v*` → `scoda-engine` (Desktop EXE + Docker)

## PyInstaller 빌드

### `ScodaDesktop.spec`

- **멀티 EXE 지원**: Desktop GUI + MCP 스폰 대상
- `scoda_engine_core` hiddenimports + datas (S-2 core 분리 대응)
- Windows 실행 시 여러 문제 순차 수정:
  - Overlay DB 경로
  - Logger 핸들러 누락 (gui.py)
  - Vendor static 파일 포함
- 02-25 (020): 멀티 사이즈 ICO 아이콘 (`scoda_engine/static/img/scoda.ico`)

### `scripts/release.py` 범용화 (S-1 Step 2, 002)

- 기존: hardcoded trilobase 참조 (`'trilobase'` 리터럴, 특정 데이터 파일 경로)
- 변경: `--name`, `--version`, `--data-dir` CLI 인수로 주입
- 약 1,183줄 삭제 (중복 제거)
- 외부 패키지에서도 재사용 가능

## bump_version.py (03-06, 038)

CLI로 모든 버전 파일을 한 번에 갱신:

```bash
python scripts/bump_version.py 0.3.2
```

갱신 대상:
1. `pyproject.toml` — `[project] version`
2. `scoda_engine/__init__.py` — `__version__`
3. `deploy/docker-compose.yml` — 이미지 태그 (03-14, 032에서 추가)

버전 불일치로 PyInstaller 빌드와 Docker 이미지가 어긋나는 문제를 원천 차단.

## MkDocs 문서 사이트 (P19, 02-26, 022)

### 구성

- **Material for MkDocs** 테마
- **다국어(EN/KO)**: i18n 접미사 방식 (`.en.md`, `.ko.md`)
- 8개 주요 문서:
  - Introduction, Installation, Quick Start
  - SCODA Specification, Manifest Reference
  - Hub Guide, CRUD Guide, Release Process
- 랜딩 페이지

### `.github/workflows/pages.yml`

- MkDocs 빌드와 Hub index 생성 통합
- 두 artifact를 하나의 GitHub Pages 사이트에 공존
- **파일명 충돌 해결** (02-26, 023): MkDocs `index.html`과 Hub `index.json`이 동일 경로에 배포되는 문제 → Hub 측을 `scoda-hub-index.json`으로 리네이밍

상세 Hub index 동작: [SCODA Hub](scoda-hub.md)

## Manual Release 워크플로우 (02-23, 010 / P13)

P13 계획에 따른 수동 릴리스 절차:

1. `git checkout main && git pull`
2. CHANGELOG.md 업데이트
3. `python scripts/bump_version.py <NEW>` (이후 추가된 편의)
4. `git commit && git tag desktop-v<NEW>`
5. `git push origin main desktop-v<NEW>`
6. GitHub Actions가 release.yml 자동 실행
7. GitHub Release / Docker Hub 확인

## 테스트 수 추이

| 날짜 | 테스트 수 | 주요 변경 |
|---|---|---|
| 02-20 | 196 | Generic fixture 전환 + MCP subprocess 수정 |
| 02-22 | 225 | Spec alignment 29 + validate_manifest 이동 |
| 02-24 | 255 | 임의 경로 로딩 5 + Hub 클라이언트 25 |
| 02-25 | 276 | Hub SSL fallback 21 |
| 03-01 | 303 | CRUD 프레임워크 27 |
| 03-14 | 303 | Multi-Package Serving (재분류) |
| 03-18 | 232* | Meta-package 9 — 별도 브랜치 측정 |

## 관련 페이지

- [scoda-engine](scoda-engine.md) — 본체
- [scoda-engine-core](scoda-engine-core.md) — `core-v*` 태그, 독립 버전
- [SCODA Hub](scoda-hub.md) — `hub-index.yml`, `pages.yml`
- [Docker 배포](docker-deployment.md) — Docker Hub CI
- FSIS2026 [배포 인프라](deployment.md) — 비교 참고

---
*Sources: P12, P13, 009, 010, P19, 022, 023, 037, 038, 032(03-14), 002(S-1 Step 2), P07.*
