# 035. 작업관리 시스템 구축

**날짜**: 2026-03-25
**버전**: 0.2.42 → 0.2.62

## 개요

관리자(Professors)가 학부생에게 논문별 작업(자료 입력, 검토)을 할당하고, 학생이 PDF를 보면서 화석산지 후보·taxa·표본 데이터를 입력하는 작업관리 시스템 구축.

## 1. Undergraduate Students 편집 제한 (v0.2.42)

- `get_user_obj()`에 `can_edit` 속성 추가 (3개 앱)
- 템플릿 26개에서 편집/삭제/추가 버튼 숨김 (`{% if user_obj.can_edit %}`)
- 뷰 30개에 `check_can_edit()` 가드 (403 Forbidden)
- CBV 6개에 `get_context_data` 오버라이드 추가

## 2. 작업할당 시스템 (v0.2.43~v0.2.58)

### 모델
- **TaskAssignment**: worker(FK User), reference(FK Reference), phase(1:자료입력/2:검토), status(6단계), assigned_by, notes, history
- **ReferenceSiteCandidate**: 논문 내 화석산지 후보 (label, 위경도, 행정구역, 비고)
- **ReferenceTaxon.site_candidate**: FK to ReferenceSiteCandidate (nullable)

### 상태 흐름
```
자료입력: 입력대기 → 입력중 → 입력완료
검토:     검토대기 → 검토중 → 검토완료
```

### 관리자 기능
- **시스템관리 > 작업할당**: 작업자별 요약 (상태별 건수) + 전체 할당 목록 (필터/페이지네이션)
- **새 작업 할당**: 좌우 분할 — 논문 목록(체크박스) / 선택된 논문 패널
  - 필터: 미할당만 / 입력완료만 / 전체
  - 논문별 PDF 유무, Taxa, Specimen 수, 작업 상태·작업자 표시
  - 검토 할당 시 입력자와 검토자가 같으면 차단
- **할당 수정/삭제**, **작업자별 상세**

### 학생 기능
- **내 작업** 메뉴 (fsisnavbar, kprnavbar에 추가) → 본인 할당 목록
- 논문 클릭 → **작업 워크스페이스**

## 3. 작업 워크스페이스 (v0.2.47~v0.2.62)

### 화면 구성
- 좌우 분할 (PDF 60% : 데이터 입력 40%)
- 상단: 논문 정보 + 단계 뱃지 + 상태 드롭다운

### PDF 뷰어 (왼쪽)
- PyMuPDF 2x 해상도 렌더링 (학생용 별도 엔드포인트)
- 폭맞춤 표시, 확대/축소/맞춤 버튼
- 페이지 이동: 이전/다음 + 직접 입력 + 키보드 방향키
- 데이터 추가 시 현재 페이지 유지 (`?page=N`)

### 데이터 입력 (오른쪽) — 3단 계층 카드
```
산지후보 카드 (Site #1, Site #2, ...)
  └─ Taxon 카드 (종명, 지층, 시대)
       └─ Specimen 행 (표본번호, Fig., 비고)
```
- 산지 미지정 taxa는 "(산지 미지정)" 카드로 별도 그룹
- 모든 항목 AJAX로 추가/삭제/in-place 편집 (페이지 새로고침 없이)
- 산지후보 추가 시 taxon 드롭다운에 실시간 반영

### 상태 자동 전환
- 워크스페이스 진입 시: 입력대기→입력중, 검토대기→검토중
- 데이터 추가 시: 대기 상태이면 자동으로 진행중으로 변경

### 완료 버튼
- 자료입력 단계: "입력 완료" → 상태를 입력완료로 변경 → 내 작업으로 이동
- 검토 단계: "검토 완료" → 상태를 검토완료로 변경 → 내 작업으로 이동

## AJAX API 목록

| URL | 기능 |
|-----|------|
| `task_api/add_taxon/<ref_pk>/` | Taxon 추가 |
| `task_api/edit_taxon/<pk>/` | Taxon 수정 |
| `task_api/delete_taxon/<pk>/` | Taxon 삭제 |
| `task_api/add_specimen/<taxon_pk>/` | Specimen 추가 |
| `task_api/edit_specimen/<pk>/` | Specimen 수정 |
| `task_api/delete_specimen/<pk>/` | Specimen 삭제 |
| `task_api/add_site_candidate/<ref_pk>/` | 산지후보 추가 |
| `task_api/edit_site_candidate/<pk>/` | 산지후보 수정 |
| `task_api/delete_site_candidate/<pk>/` | 산지후보 삭제 |
| `task_api/update_status/<pk>/` | 작업 상태 변경 |

## 신규/수정 파일

### 신규
- `kprdb/views_task.py` — 뷰 15개 + ReferenceAutocomplete
- `kprdb/templates/kprdb/task_assignment_list.html`
- `kprdb/templates/kprdb/task_assignment_add.html`
- `kprdb/templates/kprdb/task_assignment_edit.html`
- `kprdb/templates/kprdb/task_worker_detail.html`
- `kprdb/templates/kprdb/my_tasks.html`
- `kprdb/templates/kprdb/task_workspace.html`
- `kprdb/templates/kprdb/_taxon_card.html`
- `kprdb/migrations/0004~0008`

### 수정
- `kprdb/models.py` — TaskAssignment, ReferenceSiteCandidate 모델, ReferenceTaxon.site_candidate FK
- `kprdb/forms.py` — TaskAssignmentEditForm
- `kprdb/urls.py` — URL 라우트 16개 추가
- `fsis/templates/fsisnavbar.html` — 내 작업 + 작업할당 메뉴
- `kprdb/templates/kprnavbar.html` — 내 작업 + 작업할당 메뉴
- `fsis/views.py`, `ghdb/views.py`, `kprdb/views.py` — can_edit 속성
- 편집 제한 적용 템플릿 26개 + 뷰 30개
