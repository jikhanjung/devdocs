# scoda-engine (SCODA 뷰어/서버 런타임)

모든 SCODA 패키지를 렌더링하는 범용 뷰어/서버. 2026-02-19에 trilobase에서 별도 레포로 분리.

## 아키텍처

```
scoda-engine/
├── scoda_engine/
│   ├── app.js           # SPA 메인 (매니페스트 기반 뷰 렌더링)
│   ├── radial.js        # Radial tree 뷰
│   ├── tree_chart.js    # Tree/chart 뷰 (TreeChartInstance 클래스)
│   └── ...
├── launcher_gui.py      # tkinter GUI 컨트롤 패널
├── launcher_mcp.py      # MCP stdio 런처
└── serve.py             # FastAPI 서버
```

## 서버

- **FastAPI** (Flask에서 마이그레이션): 비동기 지원, OpenAPI 자동 문서
- **Pydantic** 응답 모델: 타입 안전성
- `/api/query`: 복합 엔드포인트 (여러 DB 접근)
- 동적 MCP 도구 생성: ui_queries에서 런타임 생성

## GUI 컨트롤 패널

- tkinter 기반: Start/Stop 버튼, DB 상태 표시, 자동 브라우저 열기
- 실시간 로그 뷰어 (색상 코딩)
- 구조화된 로깅 (JSON 기반)

## PyInstaller 배포

- 단일 EXE (14MB): onefile 모드
- 2-EXE 분리: Flask EXE + MCP EXE (대안 설계, 미구현)
- `--mcp-stdio`: Claude Desktop 직접 스폰 CLI 모드

## SPA (Single Page App)

- Bootstrap 5 기반
- 글로벌 검색 박스
- 브라우저 히스토리/뒤로가기 지원
- CORS 지원
- .scoda 패키지에 선택적 포함 (--with-spa 플래그)

## 범용 뷰어 기능

- 매니페스트 기반 자동 뷰 렌더링 (어떤 SCODA 아티팩트든 처리)
- 자동 검색 (auto-discovery) + fallback
- linked_table 렌더링
- redirect 기능 (Genus → rank_detail 라우팅)
- CRUD: editable_entities 선언 기반, FK autocomplete, 인라인 편집

## 뷰 타입

| 뷰 | 설명 |
|---|---|
| tree | 계층 트리 (D3.js) |
| table | 데이터 테이블 |
| detail | 상세 페이지 (linked_table 섹션) |
| chart | ICS 지질연대 차트 |
| timeline | 시간대별 분포 |
| radial | [방사형 트리](visualization.md) |
| compound | 복합 뷰 (sub-view 결합) |

## 레포 분리 (2026-02-19)

- trilobase에서 `/mnt/d/projects/scoda-engine`으로 분리
- `scoda_desktop` → `scoda_engine` 패키지 리네이밍
- 191 scoda-engine 테스트 + 66 trilobase 테스트 통과
- PyPI 배포 준비

## 관련 페이지

- [SCODA 아키텍처](scoda.md) — 패키지 규격
- [시각화](visualization.md) — 트리/차트 렌더링 상세
- [MCP 서버](mcp-server.md) — LLM 통합
- [Trilobase 개요](trilobase-overview.md)

---
*Sources: 011, 017-018, 021, 046, 050-059, 063, 066-070, 073-076, 078, P07-P19, P38-P41, P45-P49, P54-P55, P57, P62*
