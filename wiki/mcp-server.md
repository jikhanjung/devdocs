# MCP 서버 (LLM 통합)

SCODA 데이터에 LLM이 접근할 수 있도록 하는 Model Context Protocol 서버.

## 아키텍처

```
Claude Desktop ←→ [MCP stdio] ←→ scoda-engine ←→ SQLite DB
```

## MCP 도구 (14개)

ui_queries에서 런타임으로 동적 생성되는 도구 세트. Evidence Pack 빌더 패턴으로 LLM에 구조화된 응답 제공.

## 전송 모드 변천

```
Phase 22: MCP 서버 초기 구현 (14 tools, async/await)
Phase 23: SSE 모드 추가 (Starlette + Uvicorn, 포트 8081)
Phase 24: GUI에서 Flask/MCP SSE 독립 제어
Phase 25: --mcp-stdio CLI 모드 (Claude Desktop 직접 스폰)
→ SSE UI 제거, stdio 모드에 집중
```

### 최종 형태
- **stdio 모드**: `--mcp-stdio` 플래그로 Claude Desktop에서 직접 실행
- SSE 모드: 구현 후 UI에서 제거 (stdio가 더 단순)

## 동적 MCP 도구 생성 (Phase 46)

- Step 1: 복합 `/api/query` 엔드포인트 (Trilobase + PaleoCore)
- Step 2: ui_queries에서 런타임 MCP 도구 자동 생성
- Step 3: 하드코딩된 레거시 엔드포인트 제거

## Web API와의 통합

MCP 도구와 Web API가 공유 쿼리 엔진 사용 (DRY 원칙):
- 066: MCP + Web API 병합
- 통일된 쿼리 실행 경로

## 배포

- PyInstaller 단일 EXE에 MCP 서버 포함
- Claude Desktop에서 직접 스폰 가능
- GUI 컨트롤 패널에서 독립 제어

## 테스트

- asyncio 수정 (030)
- MCP subprocess 테스트
- 100+ pytest 테스트 케이스

## 관련 페이지

- [scoda-engine](scoda-engine.md) — 서버 런타임
- [SCODA 아키텍처](scoda.md) — 패키지 규격
- [Trilobase 개요](trilobase-overview.md)

---
*Sources: archive/022, 023-026, 028-030, 055-057, 066, P14-P19, P48, SCODA_MCP_Wrapping_Plan*
