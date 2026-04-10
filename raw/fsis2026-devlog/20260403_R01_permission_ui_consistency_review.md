# R01. 권한 및 인터페이스 일관성 리뷰

**날짜**: 2026-04-03  
**대상**: `fsis`, `ghdb`, `kprdb`, `evaluator`  
**주제**: 그룹 기반 페이지 권한, 추가/편집/삭제 링크 제어, UI 용어 일관성

## 1. 리뷰 목적

현재 프로젝트는 여러 시기에 개발된 앱이 병존하고 있고, 권한 제어와 UI 용어가 앱별로 조금씩 다른 방식으로 누적되어 있다.

이번 리뷰의 목표는 다음 두 가지다.

1. 현재 권한 제어가 실제로 어떤 규칙으로 동작하는지 정리한다.
2. 버튼/링크/페이지 제목/네비게이션의 용어 일관성 문제를 정리하고, 통합 기준안을 제안한다.

이 문서는 구현 문서가 아니라 리뷰 문서다. 따라서 현황, 문제점, 권장 구조, 적용 순서를 중심으로 기록한다.

---

## 2. 현재 권한 모델 요약

### 2.1 코드에 실제로 구현된 모델

현재 대부분의 앱은 아래 두 규칙을 기본으로 사용한다.

- `Undergraduate Students`가 아니면 `can_edit = True`
- `Professors`이면 `check_admin() = True`

대표 구현:

- `fsis/views.py`
- `kprdb/views.py`
- `ghdb/views.py`

즉, 현재 권한 모델은 사실상 4그룹 체계가 아니라 아래 2단 구조에 가깝다.

- 읽기 전용: `Undergraduate Students`
- 편집 가능: 그 외 전원
- 관리자: `Professors`

여기서 `Graduate Students`, `Postdoc`는 명시적으로 정의된 정책이 없고, 결과적으로 "학부생이 아니므로 편집 가능"으로 처리된다.

### 2.2 앱별 관리자 기준 차이

관리자 판단 기준도 앱마다 다르다.

- `fsis`, `kprdb`: `Professors` 그룹 기반
- `ghdb`: `is_staff` 기반

즉, 동일 사용자라도 다음이 가능하다.

- `Professors` 그룹은 있지만 `is_staff=False`이면 `fsis/kprdb` 관리자, `ghdb`에서는 비관리자
- `is_staff=True`지만 `Professors` 그룹이 없으면 `ghdb` 관리자, `fsis/kprdb`에서는 비관리자

이 불일치는 운영 중 권한 오해를 만들 가능성이 높다.

### 2.3 템플릿 노출 규칙과 서버 권한 규칙이 분리되어 있음

여러 페이지에서 버튼 노출은 `user_obj.can_edit`로 제어하지만, 실제 뷰에서는 별도의 검사 없이 `login_required`만 있는 경우가 있다.

즉, UI 상으로는 숨겨도 URL 직접 접근으로 수정 가능한 페이지가 존재한다.

---

## 3. 권한 리뷰 상세

## 3.1 핵심 진단

현재 권한 코드는 "중앙 정책"이 아니라 "앱별 관성"으로 유지되고 있다.

주요 특징:

- 그룹명 문자열 비교가 여러 파일에 직접 박혀 있음
- `can_edit`, `check_admin`, `is_staff`가 혼용됨
- 템플릿에서 버튼 숨김과 서버 단의 접근 차단이 항상 일치하지 않음
- 4개 사용자 그룹 중 2개(`Graduate Students`, `Postdoc`)는 코드상 의미가 거의 없음
- 추가/편집/삭제 페이지 링크의 노출 기준과 실제 허용 기준이 일부 페이지에서 다름

## 3.2 중요 이슈

### A. 4그룹 체계가 코드상 모델링되어 있지 않음

현재 코드에서 의미를 갖는 그룹은 사실상 다음 둘뿐이다.

- `Undergraduate Students`
- `Professors`

문제:

- `Graduate Students`, `Postdoc`는 명시 정책이 없다.
- 향후 "검토만 가능", "할당 관리 가능", "사용자 관리 불가" 같은 중간 권한 모델을 추가하기 어렵다.

권장:

- 그룹명을 직접 비교하는 대신, 프로젝트 공통 권한 함수를 두고 역할을 명시적으로 선언해야 한다.
- 예: `can_view`, `can_edit`, `can_review`, `can_manage_tasks`, `can_manage_users`, `can_view_admin_tools`

### B. 관리자 기준이 앱마다 다름

대표 사례:

- `fsis/views.py`, `kprdb/views.py`: `check_admin()`는 `Professors` 그룹만 본다.
- `ghdb/views.py`: `_check_staff()`는 `is_staff`만 본다.
- `ghdb/templates/ghdb/navbar.html`도 `user_obj.is_staff`를 사용한다.

영향:

- 사용자 관리 페이지, 시스템관리 메뉴, admin 링크 접근 범위가 앱마다 달라진다.
- 운영자가 "왜 fsis에서는 보이고 ghdb에서는 안 보이는가"를 설명하기 어렵다.

권장:

- 관리자 권한 기준을 프로젝트 전역에서 하나로 통일한다.
- 실무적으로는 아래 둘 중 하나를 택해야 한다.
  - `Professors` 그룹을 기준으로 하고 `is_staff`는 Django admin 용 내부 플래그로만 사용
  - 또는 역할 그룹과 `is_staff`를 명확히 매핑하는 사용자 생성 정책을 도입

### C. KPRDB는 템플릿 제어보다 서버 권한 제어가 느슨함

`kprdb`에는 `user_obj.can_edit`로 버튼을 숨기는 템플릿이 많지만, 실제 함수형 CRUD 뷰는 `login_required`만 있고 편집 권한 검사가 빠진 경우가 많다.

대표 예:

- `kprdb/views.py`
  - `author_add`, `author_edit`, `author_delete`
  - `journal_add`, `journal_edit`, `journal_delete`
  - `reference_add`, `reference_edit`, `reference_delete`
  - `lithounit_add`, `lithounit_edit`, `lithounit_delete`
  - `chronounit_add`, `chronounit_edit`, `chronounit_delete`
  - `scientificname_add`, `scientificname_edit`, `scientificname_delete`
  - `referencetaxon_edit`, `referencetaxonspecimen_edit`

의미:

- 학부생은 버튼은 안 보이지만, URL을 알면 수정 화면 또는 삭제 동작에 접근할 수 있다.
- 이 문제는 "UI 제한"이 아니라 "서버 권한 누수"다.

권장:

- `kprdb`의 모든 쓰기 뷰에 공통 `check_can_edit` 또는 데코레이터를 적용한다.
- 템플릿 숨김은 보조 수단으로 유지하고, 서버 차단을 기준으로 삼아야 한다.

### D. 버튼 비노출과 disabled 버튼 전략이 혼재

대표 사례:

- `fsis`와 `ghdb`는 대체로 권한 없으면 버튼을 숨긴다.
- `kprdb`는 권한 없을 때 `Edit/Delete/Add` disabled 버튼을 그대로 노출하는 페이지가 많다.

영향:

- 앱별 사용자 경험이 달라진다.
- "권한 없음"을 보여줄지, 그냥 숨길지 일관된 결정이 없다.
- 비활성 버튼은 접근성 측면에서도 의미가 애매하다. 클릭도 안 되지만 왜 안 되는지 설명이 없다.

권장:

- 정책을 통일한다.
- 개인적으로는 다음 중 하나를 선택하는 것이 좋다.
  - 기본: 숨김
  - 예외: 업무상 존재를 보여줘야 하는 기능만 disabled + 툴팁

현재 프로젝트 상태에서는 "숨김 기본, 관리 도구만 예외적 표시"가 가장 단순하다.

### E. 작업관리 권한은 사실상 `Professors` 전용 + 일반 로그인 사용자 혼합

`kprdb/views_task.py`를 보면 다음처럼 나뉜다.

- 할당 목록/할당 추가/할당 수정/할당 삭제: `check_admin(user_obj)`
- 내 작업/작업 워크스페이스/API 일부: 로그인 사용자 허용, 별도 조건 체크

문제:

- `Graduate Students`, `Postdoc`의 역할 정의가 없다.
- 현재는 "교수는 관리", "나머지 로그인 사용자는 작업자" 정도로만 구현되어 있다.

권장:

- 작업관리 권한을 편집 권한과 분리해야 한다.
- 최소한 아래 권한을 따로 둬야 한다.
  - `can_work_on_assignment`
  - `can_manage_assignments`
  - `can_review_assignment`

### F. 권한 정보가 `get_user_obj()` 호출 부작용에 의존

`get_user_obj()`는 단순 사용자 컨텍스트 생성이 아니라 활동 로그까지 저장한다.

문제:

- 템플릿용 `user_obj`를 얻기 위한 호출과 활동 로그 기록이 결합되어 있다.
- 권한 유틸을 재사용하거나 테스트하기 어렵다.
- 권한 체크를 위해 `get_user_obj()`를 호출하면 로그가 남는 구조라 관심사가 분리되지 않는다.

권장:

- 다음을 분리한다.
  - 사용자 역할 계산
  - 템플릿 컨텍스트 구성
  - 활동 로그 기록

---

## 4. 권한 구조 권장안

## 4.1 역할 정의 제안

현재 그룹을 그대로 유지하되, 코드상 의미를 명시적으로 부여한다.

### Undergraduate Students

- 조회 가능
- 본인 작업 워크스페이스 입력 가능
- 일반 마스터 데이터 직접 수정 불가
- 사용자 관리/시스템관리 불가

### Graduate Students

- 조회 가능
- 업무 데이터 입력 가능
- 작업 워크스페이스 입력 가능
- 일부 수정 가능
- 시스템관리 불가

### Postdoc

- 조회 가능
- 업무 데이터 입력/수정 가능
- 검토 권한 가능
- 작업관리 일부 가능 여부 정책 결정 필요
- 사용자 관리/시스템관리 불가 또는 제한적 허용

### Professors

- 전체 조회/입력/수정/삭제 가능
- 작업 할당/검토/사용자 관리/운영 도구 접근 가능

## 4.2 권한 함수 제안

예시:

```python
def has_group(user, name): ...
def role_names(user): ...
def can_edit_data(user): ...
def can_manage_tasks(user): ...
def can_review_tasks(user): ...
def can_manage_users(user): ...
def can_view_ops(user): ...
```

템플릿에서도 동일한 의미를 써야 한다.

예:

- `user_perm.can_edit_data`
- `user_perm.can_manage_tasks`
- `user_perm.can_manage_users`

핵심은 `can_edit` 하나로 모든 쓰기 권한을 뭉개지 않는 것이다.

## 4.3 권한 매트릭스 초안

| 기능 | UG | Grad | Postdoc | Professor |
|---|---|---|---|---|
| 목록/상세 조회 | O | O | O | O |
| 일반 CRUD | X | O | O | O |
| 작업 워크스페이스 입력 | O | O | O | O |
| 작업 검토 | X | 정책 결정 필요 | O | O |
| 작업 할당 관리 | X | X | 정책 결정 필요 | O |
| 사용자 관리 | X | X | X | O |
| 운영 도구(PDF 상태, 접속기록 등) | X | X | X | O |

주의:

- 이 표는 현재 구현이 아니라 권장 초안이다.
- 특히 `Graduate Students`, `Postdoc`의 검토/할당 권한은 운영 정책을 먼저 확정해야 한다.

---

## 5. 인터페이스 일관성 리뷰

## 5.1 전반 진단

UI는 전반적으로 Bootstrap 기반 공통 톤은 있으나, 텍스트 계층은 앱별 편차가 크다.

가장 큰 패턴은 다음 세 가지다.

- 한글/영문 혼용
- 동일 개념의 용어가 앱마다 다름
- 리스트/상세/폼/버튼 명명이 통일되어 있지 않음

## 5.2 대표 혼재 사례

### A. 페이지 제목 한글/영문 혼용

예:

- `fsis/templates/fsis/author_form.html`: `Add new author`
- `fsis/templates/fsis/chrono_list.html`: `ChronoUnit list`
- `kprdb/templates/kprdb/author_list.html`: `Author list`
- `kprdb/templates/kprdb/journal_detail.html`: `journal details`
- `kprdb/templates/kprdb/reference_detail.html`: `Reference details`
- `evaluator/templates/evaluator/evaluation_form.html`: `위험도 평가 수정/등록`

문제:

- 같은 서비스 안에서 어떤 화면은 영어, 어떤 화면은 한글이다.
- 제목 capitalization도 들쭉날쭉하다. 예: `Author list`, `journal details`

권장:

- 내부 사용자 대상 시스템이므로 한글을 기본 언어로 통일하는 것이 자연스럽다.
- 예:
  - `저자 목록`
  - `저자 등록`
  - `저자 수정`
  - `저널 상세`
  - `논문 상세`

### B. 액션 버튼 용어 혼용

현재 혼재:

- `Add` / `추가`
- `Edit` / `편집` / `수정`
- `Delete` / `삭제`
- `Save` / `저장`
- `List` / `목록`
- `History` / `변경내역`

특히 `kprdb`는 영어 액션이 많고, `fsis`/`ghdb`는 한글 액션이 많다.

권장 표준:

- 생성 진입: `등록`
- 수정 진입: `수정`
- 삭제: `삭제`
- 제출: `저장`
- 목록 복귀: `목록`
- 변경 이력: `변경내역`
- 상세 복귀/관련 화면 이동: `상세`

주의:

- 버튼 텍스트는 명사형보다 동사형이 낫다.
- 예: `Author Add`보다 `저자 등록`

### C. 동일 기능의 명칭 차이

예:

- `변경내역`, `편집기록`, `History`가 혼재
- `논문처리 현황`, `PDF 처리현황`, `ref_processing`이 섞여 있음
- `산지후보`, `화석산지`, `후보만 (미등록)` 등 용어 경계가 다소 불명확

권장:

- 용어집을 먼저 정의하고 템플릿 전체에 적용한다.

예시 용어집:

- History 계열: `변경내역`
- Activity 계열: `접속기록`
- Processing 계열: `논문 처리현황`
- Candidate 계열: `화석산지 후보`
- Registered site 계열: `화석산지`

### D. 네비게이션 용어도 앱별 차이 존재

예:

- `fsisnavbar.html`: `표본정보`, `논문정보`, `기본정보`, `시스템관리`
- `kprnavbar.html`: `논문정보`, `기본정보`, `시스템관리`
- `ghdb/navbar.html`: 단순 메뉴 + `일괄입력`

문제:

- 상위 메뉴는 비교적 통일되어 가고 있지만, 하위 메뉴 용어가 앱별로 완전히 맞춰져 있지는 않다.
- 예를 들어 사용자 메뉴에서 어떤 앱은 `내 편집기록`, 어떤 앱은 `내 정보`, 어떤 앱은 비밀번호 변경까지 제공한다.

권장:

- 공통 네비게이션 규칙을 문서화한다.
- 최소한 다음은 통일:
  - 사용자 메뉴
  - 시스템관리 메뉴
  - 목록/상세/등록/수정 흐름의 버튼 명칭

### E. 상세 페이지 액션 배치도 일관되지 않음

예:

- 어떤 페이지는 `편집/삭제/목록`
- 어떤 페이지는 `Edit/Delete/History/List`
- 어떤 페이지는 `변경내역` 버튼이 있고 어떤 페이지는 없음

권장:

상세 페이지 액션 표준:

1. `수정`
2. `삭제`
3. `변경내역` 또는 `이력`
4. `목록`

권한이 없는 경우:

- 숨김 또는 disabled + `권한 없음` 툴팁 중 한 가지로 통일

---

## 6. 우선 정리 대상

## 6.1 1순위: 서버 권한 누수 정리

가장 먼저 할 일은 `kprdb`의 쓰기 뷰를 전부 서버 단에서 막는 것이다.

대상:

- 저자 CRUD
- 저널 CRUD
- 논문 CRUD
- 학명/지층/시대 CRUD
- taxa/specimen 편집

방법:

- 공통 데코레이터 또는 헬퍼 도입
- 모든 쓰기 엔드포인트에 적용

이 작업은 UI 정리보다 먼저 해야 한다.

## 6.2 2순위: 권한 유틸 중앙화

추천 위치:

- `config/permissions.py`
- 또는 공용 앱/유틸 모듈

정리 대상:

- 그룹명 문자열 비교 제거
- `can_edit`, `check_admin`, `_check_staff` 통합
- 템플릿에서 쓰는 컨텍스트 명도 통일

## 6.3 3순위: 템플릿 액션 텍스트 통일

1차 대상:

- `kprdb`의 영어 버튼/제목
- `fsis`의 legacy 영문 제목
- `evaluator`의 `op == 'Edit'` 같은 영문 상태값 노출

빠른 효과가 크고, 리스크가 낮다.

## 6.4 4순위: 네비게이션/관리 메뉴 표준화

다음 항목을 공통 기준으로 맞춘다.

- 사용자 메뉴
- 시스템관리 메뉴
- 이력/접속기록/처리현황 계열 명칭

---

## 7. 권장 구현 방향

## 7.1 권한

추천 방식:

1. 그룹 기반 역할 정의를 중앙화한다.
2. 뷰는 반드시 서버 단에서 권한을 체크한다.
3. 템플릿은 같은 권한 객체를 사용해 버튼 노출을 맞춘다.
4. URL 직접 접근, POST 요청, AJAX 요청 모두 같은 규칙으로 막는다.

예시 구조:

```python
@login_required
@require_permission("edit_data")
def reference_edit(...):
    ...
```

또는:

```python
perm = build_user_permissions(request.user)
if not perm.can_edit_data:
    return HttpResponseForbidden(...)
```

## 7.2 UI 텍스트

권장 원칙:

- 운영자용 내부 서비스이므로 한글 우선
- 엔티티명 + 동작명으로 통일
- 버튼은 짧고 예측 가능하게 유지

예시:

- `Add` → `등록`
- `Edit` → `수정`
- `Delete` → `삭제`
- `Save` → `저장`
- `List` → `목록`
- `History` → `변경내역`
- `Reference details` → `논문 상세`
- `Author list` → `저자 목록`

---

## 8. 최종 정리

이번 리뷰의 결론은 다음과 같다.

### 권한 측면

- 현재 코드는 4그룹 체계가 아니라 사실상 2단 권한 모델이다.
- `Graduate Students`, `Postdoc`는 정책상 의미가 코드에 거의 반영되지 않는다.
- `fsis/kprdb`는 `Professors`, `ghdb`는 `is_staff`를 써서 관리자 기준이 갈라져 있다.
- 특히 `kprdb`는 템플릿 숨김에 비해 서버 단 쓰기 권한 검사가 약한 곳이 많아 우선 보완이 필요하다.

### UI 측면

- 기본 레이아웃/스타일은 어느 정도 통일됐지만, 텍스트 체계는 아직 통일되지 않았다.
- 한글/영문 혼용, 동작명 혼용, 관리 메뉴 명칭 차이가 남아 있다.
- 우선 한글 기준 용어집과 액션 버튼 표준을 먼저 정한 뒤, 공통 템플릿부터 순차 정리하는 것이 적절하다.

---

## 9. 후속 작업 제안

후속 작업은 아래 순서를 권장한다.

1. 권한 매트릭스 확정
2. 공통 권한 유틸 모듈 도입
3. `kprdb` 쓰기 뷰 서버 권한 차단 보강
4. 템플릿 권한 조건을 공통 권한 객체로 교체
5. 버튼/헤더/메뉴 용어집 적용
6. 앱별 navbar와 상세 액션 패턴 통일

필요하면 다음 문서로 이어서 작성할 수 있다.

- `R100`: 권한 매트릭스 상세 설계안
- `R101`: UI 용어집 및 템플릿 표준안
- `Pxx`: 권한 리팩터링 구현 계획
