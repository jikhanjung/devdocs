# 051. 사용자 관리 개선 및 UI 정리

**날짜**: 2026-03-30
**버전**: 0.2.98 → 0.2.115

## 1. 사용자 관리

### 비활성 사용자 정리
- 활성 유지: admin, dclee, jschoi, sclee, yjoh, invisible_admin (6명)
- 나머지 37명 → `is_active=False`, Inactive 그룹으로 이동

### 신규등록
- 네비바/로그인 페이지에 "신규등록" 버튼 추가 (kprdb, fsis, ghdb)
- 등록 폼 bootstrap5 개선 (사용자ID, 성, 이름, 이메일, 비밀번호)
- 레이블 컬럼 폭 130px로 확대
- 신규등록 사용자: `is_active=False` (비활성) + Undergraduate Students 그룹
- 등록 후 "관리자 승인 후 사용할 수 있습니다" 메시지 → 로그인 페이지

### 로그인
- 비활성 사용자 로그인 시도 → "관리자 승인 후 사용할 수 있습니다" 경고
- 비밀번호 오류 → "아이디 또는 비밀번호가 올바르지 않습니다"
- 로그인 페이지에 Django messages 표시 영역 추가

### 관리자 사용자 관리
- 사용자 목록: 상태(활성/비활성) 뱃지, 가입일 컬럼 추가, Inactive 기본 숨김
- 사용자 상세: 성/이름 분리, 그룹 뱃지, 활성 상태, 가입일, 최근 접속 표시
- 사용자 편집: `is_active` 체크박스 추가 (관리자가 활성/비활성 토글)
- 비밀번호 변경: `SetPasswordForm` → 다른 사용자 비밀번호 변경 시 기존 비밀번호 불필요
- 사용자 편집/비밀번호 변경 폼 bootstrap5 horizontal layout 적용

## 2. UI 개선

### 사용자 폼 정리
- `fsis_user_form.html`: bootstrap5 horizontal layout, 돌아가기/취소 버튼
- `user_form_admin.html`: bootstrap5, 사용자ID 읽기전용, 활성 상태 체크박스
- `user_change_password.html`: bootstrap5, 대상 사용자명 표시
- `user_register_form.html`: bootstrap5, 레이블 폭 확대

### reference_detail Taxa 그룹핑
- 산지후보(SiteCandidate)별 파란 테두리 카드로 taxa/specimen 그룹핑
- 산지 미지정 taxa는 별도 섹션
- `_taxa_table.html` include 파일로 분리 (재사용)
- SVG 인라인 아이콘 → Bootstrap Icons 정리

### 작업관리
- 새 작업 할당: 작업 단계 변경 시 자동 필터링 (자료입력→미할당, 검토→입력완료)
- 워크스페이스: PDF 업로드 시 history 기록 (`save_without_historical_record` → `save`)
- 워크스페이스: 추출 데이터 패널 하단 고정 (`mt-auto`, PDF 뷰어 하단과 정렬)

## 수정 파일

| 파일 | 작업 |
|------|------|
| `kprdb/views.py` | user_register 비활성+그룹, user_login 에러메시지, user_change_password SetPasswordForm, reference_detail 산지그룹핑, user_list_admin Inactive 필터 |
| `kprdb/views_task.py` | api_upload_pdf history 기록 |
| `kprdb/forms.py` | UserForm is_active 추가, NewUserForm 성/이름/레이블 |
| `kprdb/templates/kprdb/user_list_admin.html` | 상태/가입일 컬럼, Inactive 토글 |
| `kprdb/templates/kprdb/user_detail_admin.html` | 전체 필드 표시, 뱃지 |
| `kprdb/templates/kprdb/user_form_admin.html` | is_active 체크박스 |
| `kprdb/templates/kprdb/user_change_password.html` | bootstrap5 |
| `kprdb/templates/kprdb/user_register_form.html` | bootstrap5, 레이블 폭 |
| `kprdb/templates/kprdb/user_login_form.html` | messages, 신규등록 버튼 |
| `kprdb/templates/kprdb/reference_detail.html` | 산지별 그룹핑 |
| `kprdb/templates/kprdb/_taxa_table.html` | **신규** — taxa 테이블 include |
| `kprdb/templates/kprdb/task_assignment_add.html` | phase→show_mode 자동전환 |
| `kprdb/templates/kprdb/task_workspace.html` | 추출 패널 하단 고정 |
| `kprdb/templates/kprnavbar.html` | 신규등록 링크 |
| `fsis/templates/fsisnavbar.html` | 신규등록 버튼 |
| `fsis/templates/fsis/login.html` | 신규등록 링크 |
| `fsis/templates/fsis/fsis_user_form.html` | bootstrap5 |
| `ghdb/templates/ghdb/login.html` | 신규등록 링크 |
| `config/version.py` | 0.2.98 → 0.2.116 |

## 3. 워크스페이스 패널 분할 (v0.2.116)

오른쪽 패널을 상단/하단으로 분할하여 독립 스크롤:

- **상단** (`ws-right-top`, flex:3 = 3/4): 입력 데이터(산지/taxa/표본) + 완료 버튼
- **하단** (`ws-right-bottom`, flex:1 = 1/4): 추출 데이터 패널
- 양쪽 `overflow-y: auto`로 데이터가 많아도 각각 스크롤
- `ws-right` 자체는 `overflow: hidden`으로 전체 스크롤 방지

## 4. 작업할당 페이지 개선 (v0.3.0-0.3.1)

- 좌우 비율 7:5 → 8:4 (왼쪽 논문목록 넓힘, 오른쪽 선택목록 좁힘)
- 논문목록에 추출 정보 컬럼 추가 (좌표/Taxa/표본/SP 뱃지)
- 연도 컬럼 제거 (양쪽 모두 — short_title에 연도 포함)
- 상태 컬럼: 뱃지 + 사용자ID 두 줄 표시

## 5. 스모크 테스트 추가

- `kprdb/tests.py`: 55개 테스트 (kprdb 24 + fsis 27 + evaluator 3 + 공통 1)
- `ghdb/tests.py`: 7개 테스트
- 합계 **62개** 스모크 테스트, 모든 주요 URL에 대해 500 에러 방지
- `ReferenceSiteCandidate` import 누락 버그 발견 및 수정
- ghdb 로그인 템플릿 kprdb 네임스페이스 에러 수정

## 6. 기타 수정

- `ReferenceSiteCandidate` import 추가 (reference_detail 500 에러 수정)
- 워크스페이스 상단 연도 중복 제거
- PDF 업로드 시 history 기록 (`save_without_historical_record` → `save`)
- ghdb 로그인 페이지 신규등록 링크: `{% url %}` → 절대경로 (`/kprdb/user_register/`)

## 수정 파일 (추가분)

| 파일 | 작업 |
|------|------|
| `kprdb/tests.py` | **신규** — 55개 스모크 테스트 |
| `ghdb/tests.py` | **신규** — 7개 스모크 테스트 |
| `kprdb/templates/kprdb/task_assignment_add.html` | 추출 정보, 비율 변경, 연도 제거 |
| `ghdb/templates/ghdb/login.html` | 신규등록 링크 절대경로 |
| `config/version.py` | 0.2.98 → 0.3.1 |
