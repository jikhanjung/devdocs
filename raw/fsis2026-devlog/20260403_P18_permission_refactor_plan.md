# P18: 그룹 기반 권한 리팩터링 계획

**작성일**: 2026-04-03  
**상태**: 계획 수립

---

## 배경

현재 프로젝트는 `fsis`, `ghdb`, `kprdb`, `evaluator` 4개 앱이 공존하고 있고, 권한 제어 방식도 시기별로 누적되면서 서로 다른 규칙이 혼재해 있다.

리뷰 문서 [R01](/home/honestjung/projects/fsis2026/devlog/20260403_R01_permission_ui_consistency_review.md) 기준으로 확인된 핵심 문제는 다음과 같다.

- 권한 모델이 명목상 4그룹(`Undergraduate Students`, `Graduate Students`, `Postdoc`, `Professors`)이지만, 실제 코드에서는 2단 구조에 가깝다.
- `Graduate Students`, `Postdoc`는 코드상 명시 정책이 거의 없다.
- `fsis`/`kprdb`는 `Professors` 그룹으로 관리자 판정을 하고, `ghdb`는 `is_staff`를 사용한다.
- `kprdb`는 템플릿에서 버튼을 숨겨도 서버 단 쓰기 뷰는 `login_required`만 있는 경우가 많아 URL 직접 접근으로 수정 가능한 지점이 있다.
- 템플릿 노출 조건과 서버 권한 조건이 일치하지 않는다.

이번 작업의 목적은 권한 모델을 중앙화하고, "UI 노출"과 "서버 차단"을 동일한 기준으로 맞추는 것이다.

---

## 목표

1. **권한 정책 중앙화**
   - 그룹명 문자열 비교를 여러 파일에 흩어 놓지 않고 공통 권한 모듈로 모은다.

2. **4그룹 역할 명시화**
   - `Undergraduate Students`, `Graduate Students`, `Postdoc`, `Professors` 각각의 역할을 코드상으로 명시한다.

3. **서버 단 권한 차단 일원화**
   - 모든 쓰기 뷰, 삭제 뷰, 운영 도구 뷰, AJAX API가 같은 권한 함수 또는 데코레이터를 사용하도록 정리한다.

4. **템플릿 권한 표시 일관화**
   - 추가/편집/삭제 버튼, 관리 메뉴, 작업관리 메뉴가 동일한 권한 객체를 기준으로 노출되도록 정리한다.

5. **관리자 기준 통일**
   - `Professors` 그룹과 `is_staff` 사용 방식을 정리하고, 앱 간 관리자 기준을 통일한다.

6. **회귀 방지**
   - 핵심 권한 시나리오를 테스트로 남긴다.

---

## 범위

### 포함

- `fsis/views.py`
- `kprdb/views.py`
- `kprdb/views_task.py`
- `kprdb/views_site_candidates.py`
- `ghdb/views.py`
- `evaluator/views.py`
- 각 앱의 navbar 및 CRUD 상세/목록 템플릿
- 권한 공통 모듈 및 템플릿 컨텍스트 제공 구조
- 권한 관련 테스트 추가

### 제외

- UI 용어 전체 통일 작업
- 템플릿 문구 전체 번역
- Django admin 자체의 세부 권한 설계
- 사용자 그룹 재편성/DB 마이그레이션

이번 문서는 권한 구조 리팩터링에 집중한다. UI 용어 정리는 별도 후속 작업으로 분리한다.

---

## 현재 문제 정리

## 1. 구현 기준이 앱마다 다름

현재 주요 권한 관련 기준:

- `user_obj.can_edit = 'Undergraduate Students' not in groupname_list`
- `check_admin(user_obj) = 'Professors' in groupname_list`
- `ghdb` 관리자 기능은 `is_staff` 사용

즉, 앱별로 아래 기준이 섞여 있다.

- 그룹 기반 편집 권한
- 그룹 기반 관리자 권한
- Django `is_staff` 기반 관리자 권한

## 2. 권한이 역할별로 분리되어 있지 않음

현재는 사실상 다음 두 가지 권한만 있다.

- 편집 가능 여부
- 관리자 여부

하지만 실제 기능은 더 세분화되어 있다.

- 일반 데이터 편집
- 작업 할당 관리
- 작업 검토
- 사용자 관리
- 운영 도구 접근

이 기능들을 `can_edit`와 `check_admin` 두 개로 처리하려다 보니 중간 그룹 정책을 세우기 어렵다.

## 3. 서버 차단보다 템플릿 숨김에 의존하는 구간이 있음

특히 `kprdb`에서 다음 문제가 크다.

- 상세 페이지에서 `Edit/Delete` 버튼은 숨김 또는 disabled
- 하지만 해당 URL 뷰는 `login_required`만 있고 `can_edit` 검사가 없음

이 구조는 보안 관점에서도 좋지 않고, 운영 규칙 문서화도 어렵다.

## 4. `get_user_obj()`가 너무 많은 역할을 담당함

현재 `get_user_obj()`는 다음 역할을 동시에 한다.

- 사용자 인증 상태 확인
- 그룹 목록 계산
- `can_edit` 계산
- 활동 로그 저장
- 템플릿에 넘길 객체 구성

이 구조는 권한 로직을 재사용하기 어렵고 테스트도 불편하다.

---

## 권한 모델 제안

## 1. 역할 기반 권한 항목

공통 권한 객체는 최소한 아래 속성을 제공해야 한다.

- `can_view`
- `can_edit_data`
- `can_review_data`
- `can_work_on_assignment`
- `can_manage_assignments`
- `can_manage_users`
- `can_view_ops`
- `is_professor`
- `is_staff_user`

여기서 핵심은 `can_edit` 하나로 모든 쓰기 권한을 표현하지 않는 것이다.

## 2. 그룹별 권장 권한 매트릭스

| 권한 | Undergraduate | Graduate | Postdoc | Professors |
|---|---|---|---|---|
| 일반 조회 | O | O | O | O |
| 일반 데이터 편집 | X | O | O | O |
| 작업 워크스페이스 입력 | O | O | O | O |
| 작업 검토 | X | X 또는 O | O | O |
| 작업 할당 관리 | X | X | X 또는 O | O |
| 사용자 관리 | X | X | X | O |
| 운영 도구 접근 | X | X | X | O |

주의:

- `Graduate Students`, `Postdoc`의 검토/할당 관리 권한은 운영 정책 확정이 필요하다.
- 1차 구현에서는 안전하게 다음처럼 시작하는 것이 좋다.
  - `Graduate Students`: 편집 가능, 검토 불가, 할당 관리 불가
  - `Postdoc`: 편집 가능, 검토 가능, 할당 관리 불가
  - `Professors`: 전체 가능

이후 정책 변경이 필요하면 공통 권한 모듈만 수정하면 된다.

## 3. 관리자 기준 통일 방안

권장 기준:

- 프로젝트 내부 운영권한은 `Professors` 그룹을 기준으로 통일한다.
- `is_staff`는 Django admin 접근을 위한 내부 플래그로만 본다.
- 사용자 생성/수정 시 `Professors`이면 `is_staff=True`를 맞춰 주는 보조 정책을 둘 수 있다.

즉:

- UI와 서비스 권한: 그룹 기반
- Django admin: `is_staff`
- 둘 사이 관계: 동기화는 하되, 서비스 로직은 그룹 기반으로 판단

이 방식을 택하면 `ghdb`의 `_check_staff()`를 그룹 기반 공통 권한 함수로 대체할 수 있다.

---

## 구현 방향

## 1. 공통 권한 모듈 도입

신규 모듈 예시:

```python
config/permissions.py
```

포함 내용:

- 그룹명 상수
- 역할 계산 함수
- 권한 객체 생성 함수
- 뷰 가드 함수/데코레이터

예시 구조:

```python
GROUP_UNDERGRAD = "Undergraduate Students"
GROUP_GRAD = "Graduate Students"
GROUP_POSTDOC = "Postdoc"
GROUP_PROF = "Professors"

@dataclass
class UserPermission:
    is_authenticated: bool
    groups: set[str]
    can_view: bool
    can_edit_data: bool
    can_review_data: bool
    can_work_on_assignment: bool
    can_manage_assignments: bool
    can_manage_users: bool
    can_view_ops: bool
```

핵심 함수:

- `build_user_permissions(user)`
- `require_permission("can_edit_data")`
- `inject_user_permissions(request, user_obj)`

## 2. `get_user_obj()` 역할 축소

리팩터링 방향:

- `get_user_obj()`는 템플릿용 사용자 래퍼 또는 단순 user context만 담당
- 활동 로그 기록은 별도 함수로 분리
- 권한 계산은 공통 권한 모듈에서 수행

예:

- `build_user_permissions(request.user)` → 권한 계산
- `log_user_activity(request, source_system=...)` → 로그 기록
- `build_template_user_context(request.user, permissions)` → 템플릿 표시용 객체

## 3. 뷰 권한 데코레이터 도입

예시:

```python
@login_required
@require_permission("can_edit_data")
def author_edit(...):
    ...
```

또는:

```python
perm = build_user_permissions(request.user)
if not perm.can_edit_data:
    return HttpResponseForbidden(...)
```

정책:

- 쓰기 뷰: 반드시 서버 단 검사
- 삭제 뷰: 반드시 서버 단 검사
- AJAX API: 반드시 서버 단 검사
- 운영 도구: 반드시 서버 단 검사

## 4. 템플릿 권한 기준 통일

현재:

- `user_obj.can_edit`
- `'Professors' in user_obj.groupname_list`
- `user_obj.is_staff`

목표:

- 템플릿에서는 공통 권한 객체만 사용

예:

- `user_perm.can_edit_data`
- `user_perm.can_manage_assignments`
- `user_perm.can_manage_users`
- `user_perm.can_view_ops`

이렇게 바꾸면 navbar, 상세 페이지 액션, 목록의 Add 버튼이 같은 기준을 쓰게 된다.

---

## 단계별 실행 계획

## Phase 1. 권한 모듈 도입

목표:

- 중앙 권한 정의를 먼저 만든다.

작업:

1. `config/permissions.py` 신규 생성
2. 그룹 상수 정의
3. `UserPermission` dataclass 정의
4. `build_user_permissions(user)` 구현
5. 공통 권한 검사 헬퍼 구현
6. 템플릿에 전달할 `user_perm` 객체 규약 결정

완료 기준:

- 어떤 뷰에서도 동일 함수로 권한 판단 가능
- 테스트 없이도 기존 코드에서 병행 사용 가능

## Phase 2. `fsis` 권한 기준 이관

목표:

- 기존 `check_can_edit`, `check_admin`를 공통 모듈 기반으로 바꾼다.

작업:

1. `fsis/views.py`의 `check_admin`, `check_can_edit` 내부를 공통 권한 함수 사용으로 교체
2. `get_user_obj()`에서 `can_edit` 직접 계산 제거 또는 호환 계층으로 유지
3. `fsisnavbar.html`의 교수 판별과 사용자 메뉴 권한을 `user_perm`으로 변경
4. 상세/목록 템플릿의 추가/편집/삭제 버튼 조건 변경

완료 기준:

- `fsis`는 기존 동작을 유지하면서 공통 권한 체계를 사용

## Phase 3. `ghdb` 관리자 기준 통일

목표:

- `ghdb`의 `is_staff` 의존을 제거하고 프로젝트 공통 권한 체계로 맞춘다.

작업:

1. `ghdb/views.py`의 `_check_staff()` 제거 또는 래핑
2. 사용자 관리 뷰를 `can_manage_users` 기준으로 변경
3. navbar의 `user_obj.is_staff` 조건을 `user_perm.can_manage_users`로 변경
4. `is_staff`는 Django admin 접근 전용으로만 남기되, 사용자 관리 폼 저장 시 정책적으로 동기화할지 결정

완료 기준:

- `ghdb`의 관리자 메뉴와 관리자 뷰가 `fsis/kprdb`와 같은 기준을 사용

## Phase 4. `kprdb` 서버 권한 누수 차단

목표:

- 가장 위험도가 큰 구간부터 서버 단 권한 차단을 보강한다.

우선 대상:

- 저자 CRUD
- 저널 CRUD
- 논문 CRUD
- 학명 CRUD
- 지층/시대 CRUD
- `referencetaxon_edit`
- `referencetaxonspecimen_edit`

작업:

1. 각 함수형 뷰에 `can_edit_data` 검사 추가
2. 관리자 전용 뷰에 `can_manage_users` 또는 `can_view_ops` 적용
3. `pdf_status`, `site_candidates`, 접속기록, 사용자 관리 등의 운영 도구 권한 재정리

완료 기준:

- 버튼 숨김과 상관없이 URL 직접 접근 시 올바르게 403 또는 redirect 처리

## Phase 5. `kprdb` 작업관리 권한 분리

목표:

- `can_edit_data`와 별개로 작업관리 권한을 분리한다.

작업:

1. `views_task.py`에서 `check_admin` 의존 구간을 역할별 권한으로 변경
2. 작업자 전용 기능과 관리자 전용 기능 분리
3. 향후 `Postdoc` 검토 권한 확장 가능하도록 구조화

권장 1차 정책:

- `can_work_on_assignment`: 모든 로그인 사용자 중 4개 그룹 소속자
- `can_manage_assignments`: `Professors`
- `can_review_data`: `Postdoc`, `Professors`

완료 기준:

- 작업관리 시스템이 일반 CRUD와 별도 권한 축을 사용

## Phase 6. 템플릿 전면 정리

목표:

- UI 노출과 서버 권한을 같은 기준으로 맞춘다.

작업:

1. navbar
2. 목록 페이지 Add 버튼
3. 상세 페이지 Edit/Delete/History/List 버튼
4. 운영 도구 메뉴
5. disabled 버튼 전략 정리

권장 방침:

- 권한이 없으면 기본적으로 숨김
- 필요 시 disabled + tooltip을 예외적으로 사용

완료 기준:

- 템플릿 어디에서도 그룹명 문자열 직접 비교를 하지 않음

## Phase 7. 테스트 추가

목표:

- 권한 회귀를 방지한다.

우선 테스트 시나리오:

1. `Undergraduate Students`
   - 목록/상세 조회 가능
   - 주요 수정 URL POST/GET 접근 시 차단
   - 작업 워크스페이스는 허용 범위 내에서 접근 가능

2. `Graduate Students`
   - 일반 CRUD 가능
   - 사용자 관리/운영 도구 접근 불가

3. `Postdoc`
   - 일반 CRUD 가능
   - 검토 기능 허용 여부 테스트
   - 사용자 관리/운영 도구 접근 불가

4. `Professors`
   - 전 기능 접근 가능

5. 비로그인 사용자
   - 로그인 필요 페이지 redirect

완료 기준:

- 최소 smoke 수준의 권한 테스트 세트 확보

---

## 파일별 예상 수정 범위

### 신규

- `config/permissions.py`

### 주요 수정

- `fsis/views.py`
- `ghdb/views.py`
- `kprdb/views.py`
- `kprdb/views_task.py`
- `kprdb/views_site_candidates.py`
- `evaluator/views.py`
- `fsis/templates/fsisnavbar.html`
- `kprdb/templates/kprnavbar.html`
- `ghdb/templates/ghdb/navbar.html`
- 각 앱의 목록/상세 템플릿 다수
- 관련 테스트 파일

---

## 설계 원칙

1. **서버 차단 우선**
   - 템플릿 숨김은 보조 수단이다.

2. **그룹명 문자열 직접 비교 금지**
   - 템플릿과 뷰에서 반복하지 않는다.

3. **권한 이름은 기능 중심**
   - `admin` 대신 `can_manage_users`, `can_view_ops`처럼 목적이 드러나야 한다.

4. **호환 계층 허용**
   - 1차 리팩터링에서는 `user_obj.can_edit`를 잠시 유지하되, 내부 구현을 공통 권한 모듈에 위임해도 된다.

5. **작업관리 권한과 일반 CRUD 권한 분리**
   - 한 축으로 합치지 않는다.

6. **`is_staff`는 서비스 권한 기준으로 직접 사용하지 않음**
   - Django admin 전용으로 축소한다.

---

## 리스크

## 1. 기존 운영 습관과 충돌 가능

현재는 사실상 "학부생 외 전원 편집 가능" 모델이라, `Graduate Students`와 `Postdoc`의 세부 권한을 바꾸면 실제 운영 흐름에 영향이 있을 수 있다.

대응:

- 1차는 기존 편집 가능 범위를 유지하되, 관리자/검토/운영 도구만 분리

## 2. 템플릿 다수가 영향 범위

권한 버튼 조건이 여러 템플릿에 분산되어 있어 일부 페이지가 누락될 수 있다.

대응:

- navbar → 목록 → 상세 → 운영 도구 순으로 체크리스트화

## 3. 활동 로그와 권한 계산 분리 시 부수효과 가능

`get_user_obj()` 정리가 지나치게 크게 번지면 권한 작업 범위를 넘을 수 있다.

대응:

- 1차는 호환 계층 유지
- 활동 로그 분리는 최소 변경으로 시작

---

## 완료 기준

- [ ] 공통 권한 모듈 도입
- [ ] 4개 그룹 권한 규칙 코드상 명시화
- [ ] `ghdb` 관리자 기준 통일
- [ ] `kprdb` 주요 쓰기 뷰 서버 권한 차단 완료
- [ ] 작업관리 권한 분리
- [ ] navbar 및 주요 CRUD 템플릿 권한 조건 통일
- [ ] 권한 테스트 추가
- [ ] 그룹명 문자열 직접 비교가 뷰/템플릿에서 제거 또는 최소화

---

## 권장 실행 순서

1. `config/permissions.py` 초안 구현
2. `fsis` 권한 함수 이관
3. `ghdb` 관리자 기준 통일
4. `kprdb` 쓰기 뷰 서버 차단 보강
5. `kprdb` 작업관리 권한 분리
6. 템플릿 조건식 정리
7. 테스트 추가

---

## 후속 문서

이 계획이 승인되면 다음 문서 또는 작업으로 이어질 수 있다.

- `R02`: 권한 매트릭스 확정 리뷰
- 구현 devlog: Phase별 실제 적용 기록
- UI 용어 표준화 계획 문서
