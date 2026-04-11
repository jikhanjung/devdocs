# SCODA Hub (정적 패키지 레지스트리)

SCODA Hub은 GitHub Pages에 호스팅되는 **정적 패키지 레지스트리**. 패키지 repo의 GitHub Releases를 스캔하여 `scoda-hub-index.json`을 생성·배포하고, Desktop GUI나 Docker 빌드 파이프라인이 이를 사용해 `.scoda` 패키지를 자동 다운로드·갱신한다.

## 설계 목표 (P15, P16, P17)

- **서버 없음**: 오직 GitHub Pages + GitHub Releases로 운영
- **순수 stdlib 클라이언트**: 외부 의존성 없이 Desktop/Docker에서 동일 코드 동작
- **기관 네트워크 호환**: SSL 프록시/자가서명 인증서 환경 대응
- **선언적 의존성**: SemVer range 기반 해결

## 아키텍처

```
┌────────────────────┐
│ Package Repos      │ (jikhanjung/trilobase, ...)
│  - GitHub Releases │    release 자산: .scoda + .manifest.json
└────────┬───────────┘
         │ GitHub REST API
         ▼
┌────────────────────┐
│ scoda-engine repo  │
│  scripts/          │    generate_hub_index.py
│  generate_hub_...  │    (매주 월요일 cron + 수동 dispatch)
└────────┬───────────┘
         │ gh-pages deploy
         ▼
┌────────────────────┐
│ GitHub Pages       │
│  scoda-hub-...     │    scoda-hub-index.json
│  index.json        │    (+ MkDocs 사이트 공존)
└────────┬───────────┘
         │ HTTPS GET
         ▼
┌────────────────────┐
│ hub_client.py      │
│  - fetch           │    Desktop GUI / Docker fetch_packages.py
│  - compare         │
│  - download        │
│  - resolve deps    │
└────────────────────┘
```

## Hub Index 생성 (02-24, 013 / P15)

### `hub/sources.json` (수동 관리, git 커밋)

```json
[
  { "repo": "jikhanjung/trilobase", "type": "github_releases" }
]
```

### `scripts/generate_hub_index.py`

- **순수 stdlib**: `urllib.request`만 사용
- `GITHUB_TOKEN` 환경변수 지원 (rate limit 회피, Actions 자동 제공)
- **2단계 수집 전략**:
  1. **Strategy 1**: 릴리스에 `*.manifest.json` 자산이 있으면 다운로드 + 파싱 (메타데이터 풍부)
  2. **Strategy 2**: `.scoda` 파일명 패턴 fallback (`name-version.scoda`)
- 멀티 패키지 릴리스 지원 (하나의 릴리스에 여러 `.scoda` 포함)
- SemVer 정렬로 latest 결정

CLI:
```bash
python scripts/generate_hub_index.py                           # 생성
python scripts/generate_hub_index.py --dry-run                 # 미리보기
python scripts/generate_hub_index.py --all                     # (초기) 전체 릴리스
python scripts/generate_hub_index.py --output hub/scoda-hub-index.json
```

### `.github/workflows/hub-index.yml`

- **트리거**: `workflow_dispatch` (수동) + `schedule` (매주 월요일 00:00 UTC)
- 권한: `contents: read`, `pages: write`, `id-token: write`
- 동시성: `group: pages`
- Job 1 (generate): `generate_hub_index.py` → Pages artifact 업로드
- Job 2 (deploy): `actions/deploy-pages@v4`

### 파일명 변경 (02-26, 023)
- `index.json` → `scoda-hub-index.json`
- 이유: MkDocs 사이트(`index.html`)와 GitHub Pages 루트에서 공존하기 위함
- MkDocs와 통합 워크플로우는 `pages.yml`에서 관리

### latest-only (03-11, 042)
- 초기엔 `--all`로 전체 릴리스 수집 → 인덱스가 불필요하게 비대
- 이후 latest 릴리스만 포함하도록 변경, `--all` 플래그 제거

## Hub Client (02-24, 014)

`scoda_engine/hub_client.py` — 순수 stdlib 모듈:

| 함수 | 설명 |
|---|---|
| `fetch_index(url)` | Hub index JSON GET + 파싱 |
| `compare(local, remote)` | 로컬 패키지 vs Hub 패키지 버전 비교 (SemVer) |
| `download(entry, dest)` | `.scoda` 다운로드 + SHA-256 검증 |
| `resolve(entry, registry)` | dependency 재귀 해결 (SemVer range) |

### Desktop GUI 통합
- Hub 섹션 (패키지 목록 + 상태 뱃지)
- 백그라운드 체크 (비동기 thread)
- 프로그레스 바 (다운로드 진행률)
- SHA-256 검증 실패 시 거부
- Dependency UI + 다운로드 확인 다이얼로그 (P17)

### 버그 수정 (02-24, 016)
- 로컬 패키지와 Hub 버전이 모두 존재할 때 비교 로직 누락 → 항상 다운로드되던 문제 수정

## SSL Fallback (02-25, 019)

기관 네트워크(프록시, 자가서명 인증서) 환경에서 SSL 검증 실패 대응:

1. 기본: 시스템 CA로 검증
2. 실패 시: **Windows 인증서 저장소**(`certifi-win32` 대체)에서 재시도
3. 사용자 승인 시: SSL 검증 비활성화 옵션 (`HubSSLError` 예외 → GUI 다이얼로그)
4. 설정 저장: `ScodaDesktop.cfg` (`hub_ssl_verify = false`)
5. 21개 테스트 추가 (276 tests)

Docker 빌드 환경에서도 `SCODA_HUB_SSL_VERIFY=0` 환경변수로 동일 동작 지원 (03-06, 035).

## Hub Manifest Spec (02-25, 017)

- `{package_id}-{version}.manifest.json` 파일명 규칙 문서화
- 필드: `name`, `version`, `sha256`, `size`, `dependencies`, `download_url`, ...
- GitHub Release 자산으로 업로드 → Strategy 1에서 우선 수집

## 주기적 / 온디맨드 동기화 (03-11, 043)

서버 재시작 없이 Hub 갱신:

- **주기적 daemon sync**: 서버 프로세스 내 백그라운드 태스크 (기본 주기 설정 가능)
- **온디맨드 API**: `POST /api/hub/sync` 엔드포인트
- **UI 통합** (03-12, P27):
  - Navbar `↻` 버튼 (프로덕션 웹 뷰어)
  - Desktop GUI "Check Hub" 버튼

## 빌드 타임 다운로드 (Docker, 03-02, 030)

`deploy/fetch_packages.py`: Docker 빌드 시 Hub index에서 최신 `.scoda`를 자동 다운로드하여 이미지에 포함. `SCODA_HUB_SSL_VERIFY=0` 플래그로 SSL 우회.

상세: [Docker 배포](docker-deployment.md)

## 인증서 / EXE 아이콘 (02-25, 020)

별도지만 동일 세션에서 처리: EXE 및 GUI tkinter 윈도우에 멀티 사이즈 ICO 아이콘 적용.

## 관련 페이지

- [scoda-engine](scoda-engine.md) — Hub 클라이언트 사용
- [SCODA 아키텍처](scoda.md) — `.manifest.json`, dependency, SemVer
- [Docker 배포](docker-deployment.md) — `fetch_packages.py` 빌드 타임 사용
- [릴리스 워크플로우](scoda-engine-release.md) — hub-index.yml, pages.yml

---
*Sources: P15, P16, P17, 013-016, 017, 019, 023, 042(03-11), 043, P27, 035.*
