# 2026-04-10 — Graphify 지식 그래프 생성

## 작업 요약

`graphify` 도구를 사용하여 devdocs 리포지토리의 devlog 코퍼스를 지식 그래프로 변환했다.
두 프로젝트(fsis2026, trilobase)에 대해 각각 그래프를 생성하고, 쿼리 탐색까지 수행했다.

## 수행 내역

### 1. fsis2026-devlog 그래프 생성 (89 files)

- **입력**: `raw/fsis2026-devlog/` 89개 문서 (~42K words)
- **추출**: 4개 서브에이전트 병렬 투입 (각 ~22파일)
- **결과**: 201 nodes, 141 edges, 70 communities
- **God Nodes**: Reference Processing Pipeline P14 (6 edges), MapLibre Migration (5), Permission Review (5)
- **토큰 절감**: 118.7x (쿼리당 ~472 tokens vs 전체 ~56K tokens)

### 2. fsis2026 그래프 쿼리 테스트

- 질문: "OCR에서 Claude 추출까지 어떻게 연결되지?"
- BFS 탐색으로 17 노드 서브그래프 추출
- 4단계 파이프라인 발견: RunPod OCR → extract_from_ocr.py → claude_augment.py → _best_extract_path()
- 3개 커뮤니티 경계 횡단 확인

### 3. trilobase-devlog 그래프 생성 (251 files)

- **입력**: `raw/trilobase-devlog/` 251개 문서 (~192K words)
- **추출**: 12개 서브에이전트 병렬 투입 (각 ~21파일)
- **결과**: 617 nodes, 740 edges, 42 communities
- **God Nodes**: Trilobase Database (18 edges), Classification Profile (16), SCODA Package Format (13)
- **토큰 절감**: 84.8x (쿼리당 ~3,018 tokens vs 전체 ~256K tokens)
- **주요 발견**:
  - Assertion-Centric Model과 User Annotations가 독립적으로 설계되었지만 동일한 "주장 저장" 패턴
  - Radial Tree / Side-by-Side Tree / Animated Morphing이 하나의 시각화 스펙트럼

## 생성된 파일

```
graphify-out/
  graph.html          - 인터랙티브 그래프 (브라우저에서 열기)
  graph.json          - 그래프 데이터 (GraphRAG 호환)
  GRAPH_REPORT.md     - 감사 보고서 (God Nodes, Surprises, Suggested Questions)
  cost.json           - 토큰 사용 추적 (2 runs)
```

## 기술 노트

- `graphify` (graphifyy PyPI 패키지) 사용
- 서브에이전트에는 `sonnet` 모델 사용 (비용 최적화)
- 각 엣지에 EXTRACTED/INFERRED/AMBIGUOUS 신뢰도 태그 부착
- hyperedge로 3+ 노드 그룹 관계 캡처 (파이프라인, 스프린트 등)
- 결과 그래프는 `/graphify query` 명령으로 지속적 쿼리 가능
