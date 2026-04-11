# Multi-Package Serving

scoda-engine v0.3.0 (2026-03-14, P29)에서 추가된 기능. 단일 FastAPI 인스턴스가 여러 `.scoda` 패키지를 **URL prefix 기반**으로 동시에 서빙한다. 프로덕션 Docker 배포에서 여러 도메인 패키지(trilobase, brachiobase, ...)를 하나의 컨테이너로 제공하기 위해 설계되었다.

## 배경

이전까지 scoda-engine은 프로세스 당 단일 패키지 서빙 모델이었다:
- `serve.py --scoda-path trilobase-0.2.2.scoda`
- `/api/taxa`, `/api/query/...` — 패키지 컨텍스트가 암묵적

여러 패키지를 제공하려면 컨테이너/프로세스를 각각 띄워야 했다. Multi-Package Serving은 이를 한 프로세스에 통합한다.

## URL 라우팅

| 경로 | 설명 |
|---|---|
| `GET /` | 랜딩 페이지 (패키지 카드 그리드) |
| `GET /api/packages` | 등록된 패키지 목록 JSON |
| `GET /{package}/` | 패키지별 프론트엔드 페이지 |
| `GET /api/{package}/manifest` | 패키지 매니페스트 |
| `GET /api/{package}/query/...` | 패키지별 쿼리 |
| `GET /api/{package}/entities/...` | CRUD |
| `GET /api/...` | **Legacy fallback** (단일 패키지 호환) |

## 구현 (P29, 035)

### APIRouter with prefix

FastAPI `APIRouter(prefix="/api/{package}")`를 사용하여 모든 기존 엔드포인트를 prefix 아래로 재등록. 기존 경로는 동일한 라우트 함수를 재사용.

### `PackageRegistry.register_db()` + `_set_paths_for_testing()`

- 기존 `PackageRegistry.scan(dir)`은 디렉토리 내 모든 `.scoda`를 자동 등록
- 신규 `register_db(name, path)`: 단일 패키지 수동 등록
- `_set_paths_for_testing()`: 테스트에서 레지스트리 재초기화용 헬퍼

### Landing Page (`templates/landing.html`, 036)

- **D3 force-directed 배경 애니메이션** (축소된 레이아웃, 시각적 장식)
- 패키지 카드 그리드
- 카드 클릭 → `/{package}/`로 네비게이트
- Home 브레드크럼 (🏠 SCODA / PackageName)

### 프론트엔드 API_BASE 주입 (035)

프론트엔드 SPA가 동작하려면 자신이 어느 패키지 컨텍스트인지 알아야 함:

```html
<!-- templates/index.html -->
<script>
  window.API_BASE = '/api/{{ package_name }}';
</script>
```

- JS 측 `resolveApiUrl(path)` 헬퍼로 모든 fetch 경로를 prefix 포함 URL로 변환
- `API_BASE = '/api/trilobase'`이면 `resolveApiUrl('/query/taxa')` → `/api/trilobase/query/taxa`

### Legacy Fallback Router (036)

하위 호환성을 위해 `/api/...`(prefix 없음) 경로는:
- 등록된 패키지가 1개면 → 해당 패키지 컨텍스트로 라우팅
- 2개 이상이면 → 410 Gone + 안내 응답

Desktop GUI 및 기존 클라이언트와의 호환 유지.

## Top-level 패키지 자동 리다이렉트 (03-18, 043)

Meta-package 도입 후 추가된 개선 (→ [meta-package.md](meta-package.md)):

- `GET /` 접속 시 모든 등록 패키지의 `dependencies`를 분석
- **Top-level 패키지 판별**: 다른 패키지의 dependency로 등장하지 않는 패키지
- Top-level이 **1개**면 → 자동 302 리다이렉트 (`/paleobase/` 등)
- Top-level이 여럿이면 → Landing에 top-level만 표시 (dependency 전용 패키지는 숨김)

예: `paleobase.scoda` + `trilobita.scoda` + `brachiopoda.scoda` 로드 시 → paleobase만 top-level → `/` 접속 시 `/paleobase/` 자동 리다이렉트.

## 테스트 및 버전

- v0.3.0, 303 tests passing
- 모든 기존 단일 패키지 테스트가 prefix 경로로 재-pass
- 통합 테스트: 2개 패키지 등록 후 각 prefix로 동시 쿼리 검증

## 관련 페이지

- [scoda-engine](scoda-engine.md) — FastAPI 본체
- [Meta-Package](meta-package.md) — top-level 리다이렉트, 합성 뷰
- [Docker 배포](docker-deployment.md) — 프로덕션 multi-package 사용 사례
- [SCODA 아키텍처](scoda.md) — dependency, manifest

---
*Sources: P29, 035, 036 (03-14), 043 (03-18).*
