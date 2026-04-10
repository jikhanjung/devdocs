# Source Processing Status

`raw/` 하위 각 디렉토리의 인제스트 처리 현황.

| 디렉토리 | 파일 수 | 소스 날짜 범위 | 마지막 처리 시각 (KST) | 처리 범위 | 상태 |
|---|---|---|---|---|---|
| `raw/fsis2026-devlog/` | 89 | 2026-02-24 ~ 2026-04-10 | 2026-04-10 23:05 | 전체 (89/89) | **완료** |
| `raw/trilobase-devlog/` | 103 + 150 (archive) | 2026-02-04 ~ 2026-03-18 | 2026-04-10 23:30 | 전체 (253/253) | **완료** |
| `raw/scoda-engine-devlog/` | 97 | 2026-02-19 ~ 2026-03-18 | — | 미처리 (0/97) | **대기** |

> **다음 인제스트 시**: 마지막 처리 시각 이후에 추가/수정된 파일만 처리하면 됩니다.
> `find raw/{dir} -newer wiki/source-status.md -name '*.md'` 로 대상 파일을 확인할 수 있습니다.

## 처리 이력

### fsis2026-devlog
- **2026-04-10 23:05 KST**: 초기 배치 인제스트. 89개 파일 전량 처리. 11개 wiki 페이지 생성 (overview, kprdb, ghdb, evaluator, pdf-pipeline, task-management, map-system, deployment, data-import, ui-design, django-migration).

### trilobase-devlog
- **2026-04-10 23:30 KST**: 초기 배치 인제스트. 253개 파일 전량 처리 (main 103 + archive 150). 9개 wiki 페이지 생성 (trilobase-overview, scoda, scoda-engine, assertion-model, taxonomy-data, visualization, mcp-server, packages, treatise-extraction).

### scoda-engine-devlog
- 미처리. 97개 파일 대기 중 (2026-02-19 ~ 2026-03-18).
