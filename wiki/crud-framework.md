# CRUD 프레임워크 (Manifest-Driven)

scoda-engine의 **제네릭 CRUD 시스템**. 도메인 엔티티별 코드를 작성하지 않고, 매니페스트 선언만으로 Create/Read/Update/Delete를 제공한다. 2026-03-01에 P21 계획에 따라 구현되었으며, Admin/Viewer 모드 분리 + overlay DB 저장 Preferences API와 함께 동작한다.

## 설계 원칙 (P21)

1. **선언적**: 엔티티 구조를 Python 코드가 아닌 manifest 스키마로 정의
2. **Parameterized SQL**: SQL 인젝션 방지, 동적 WHERE 절 조합
3. **Core 호환**: `scoda_engine_core.validate_manifest`로 스키마 검증
4. **Viewer vs Admin 모드 분리**: 읽기 전용 뷰어는 쓰기 API 미노출
5. **Overlay DB에 저장**: 정전 DB(read-only) 규칙 유지

## 컴포넌트

### `entity_schema.py`

엔티티 구조 정의 데이터클래스:

```python
@dataclass
class FieldDef:
    name: str
    type: str           # "int" | "text" | "real" | "bool" | ...
    required: bool = False
    readonly_on_edit: bool = False   # create 시엔 허용, edit에선 숨김
    unique: bool = False
    fk: Optional[str] = None         # "other_entity.id" 형식
    default: Any = None
    ...

@dataclass
class EntitySchema:
    name: str                        # 테이블명
    display_name: str                # UI 라벨
    fields: List[FieldDef]
    search_fields: List[str]
    unique_constraints: List[List[str]]
    hooks: dict                      # before_insert, after_update, ...
```

입력 검증:
- 타입 체크
- `required` 필드 누락 감지
- FK 참조 유효성 (존재 검사)
- Unique 제약 사전 확인

### `crud_engine.py`

실제 SQL 실행 엔진:

| 메서드 | 설명 |
|---|---|
| `list_entities(schema, filters, limit, offset)` | 파라미터화 WHERE + LIMIT |
| `get_entity(schema, id)` | 단일 조회 |
| `create_entity(schema, data)` | `before_insert` 훅 → INSERT → `after_insert` |
| `update_entity(schema, id, data)` | `readonly_on_edit` 필드 제외 → UPDATE |
| `delete_entity(schema, id)` | FK 참조 확인 → DELETE |
| `search_entities(schema, query)` | `search_fields` 대상 LIKE 검색 |

- 모든 SQL은 `?` 바인딩 파라미터 사용
- FK 검증: `fk = "taxon.id"` 선언 시 INSERT 전에 `SELECT 1 FROM taxon WHERE id = ?` 확인
- Unique 제약: 동일 컬럼 조합이 이미 존재하는지 확인
- 훅: `before_insert`, `after_insert`, `before_update`, `after_update`, `before_delete`

## REST API (10 endpoints)

```
GET    /api/entities/{entity_name}                  # list
GET    /api/entities/{entity_name}/{id}             # get
POST   /api/entities/{entity_name}                  # create
PUT    /api/entities/{entity_name}/{id}             # update
DELETE /api/entities/{entity_name}/{id}             # delete
GET    /api/search/{entity_name}?q=...              # search
GET    /api/entities/{entity_name}/schema           # schema JSON
GET    /api/entities/{entity_name}/autocomplete/{field}  # FK autocomplete
```

응답은 Pydantic 모델 (FastAPI 자동 OpenAPI).

## Manifest에서 선언

```json
{
  "editable_entities": [
    {
      "name": "assertion",
      "display_name": "Classification Assertion",
      "fields": [
        { "name": "id", "type": "int", "readonly_on_edit": true },
        { "name": "taxon_id", "type": "int", "required": true, "fk": "taxon.id" },
        { "name": "parent_id", "type": "int", "fk": "taxon.id" },
        { "name": "opinion_type", "type": "text", "required": true },
        { "name": "notes", "type": "text" }
      ],
      "search_fields": ["notes"],
      "unique_constraints": [["taxon_id", "opinion_type", "parent_id"]]
    }
  ]
}
```

## 프론트엔드 UI (03-01, 028)

- **Detail 뷰**: Edit / Delete 버튼 (Admin 모드에서만 표시)
- **FK Autocomplete**: 타이핑 시 `/api/entities/{entity}/autocomplete/{field}?q=` 호출
- **`readonly_on_edit` 필드**: Create 폼에서 입력 가능, Edit 폼에서 숨김
- **Inline editing**: 테이블 셀 더블클릭 → 직접 편집
- **Validation errors**: 서버 응답 에러를 필드 별로 표시

## Preferences API (03-01, 027)

CRUD 프레임워크와 같은 세션에서 구현된 Preferences 시스템:

- **문제**: localStorage에 저장되던 사용자 설정(선택한 profile, watch list, UI 상태)이 브라우저/세션 경계로 분산
- **해결**: Overlay DB(`user_annotations` 파일)에 `preferences` 테이블 생성
- API:
  ```
  GET  /api/preferences/{key}
  PUT  /api/preferences/{key}
  DELETE /api/preferences/{key}
  ```
- 프론트엔드: `localStorage.getItem/setItem`을 전부 Preferences API 호출로 교체
- localStorage 완전 제거 (한 가지 진실의 원천)

## Admin / Viewer 모드 (03-01, 028)

- **Viewer 모드** (기본): 쓰기 API 비활성화, UI에서 Edit/Delete 버튼 숨김
- **Admin 모드**: 전체 CRUD 가능
- 모드 전환: 서버 시작 플래그 (`--admin`) 또는 GUI 토글
- 프로덕션 Docker(`serve_web.py`)는 기본적으로 Viewer 모드 (MCP opt-in과 동일 원리)

## 테스트

- `tests/test_crud.py` — 27개 테스트 (03-01 세션에서 추가)
- 303 tests passing
- Generic fixture (S-1)와 호환되는 범용 엔티티 스키마로 작성

## 관련 페이지

- [scoda-engine](scoda-engine.md) — CRUD 서빙
- [SCODA 아키텍처](scoda.md) — `editable_entities` manifest 필드
- [Docker 배포](docker-deployment.md) — Viewer 모드 프로덕션 배포
- [scoda-engine-core](scoda-engine-core.md) — validate_manifest 스키마 검증

---
*Sources: P21, 026-028 (2026-03-01).*
