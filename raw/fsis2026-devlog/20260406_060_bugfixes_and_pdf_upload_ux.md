# 060. 대시보드 버그수정, PDF 업로드 UX 개선, 이력 상세 CSS 수정

**날짜**: 2026-04-06  
**버전**: 0.3.18 → 0.3.23

---

## 1. 빌드 자동화 스크립트 (`deploy/build.sh`)

테스트 → 버전 bump → Docker 빌드 → 푸시를 한 번에 처리하는 스크립트 추가.

```bash
./deploy/build.sh X.Y.Z
```

- kprdb (56개) + ghdb (7개) 테스트 순차 실행
- 테스트 실패 시 `set -e`로 즉시 중단, 이후 단계 진행 안 함
- 성공 시 `config/version.py` 자동 bump → git commit → 빌드 → 푸시

---

## 2. 대시보드 비로그인 500 에러 수정 (0.3.19)

**원인**: `fsis:index` 뷰에 `@login_required` 없이 `TaskAssignment.objects.filter(worker=request.user)` 호출 → `AnonymousUser`는 Django FK 쿼리에 사용 불가 → `ValueError`

**수정**: `request.user.is_authenticated` 체크 후 조건부로 `my_task_stats` 계산.

**회귀 테스트 추가**: `FsisSmokeTests.test_index_anonymous` — 비로그인 상태로 `/fsis/` 접근 시 500이 나지 않는지 확인.

---

## 3. PDF 확보 업로드 페이지 UX 개선 (`/kprdb/my_pdf_uploads/`)

### 3.1 상태 변경 시 행 제거 (0.3.20)

상태 드롭다운 변경 후 완료 버튼 클릭 시, 현재 URL 필터(`?status=finding_pdf` 등)와 다른 상태로 저장되면 해당 행을 0.3초 페이드아웃 후 제거.

### 3.2 완료 버튼으로 명시적 저장 (0.3.20)

기존: 드롭다운 변경 즉시 서버 저장 (auto-save)  
변경: 드롭다운 변경은 로컬만, **완료 버튼** 클릭 시에만 서버 저장.

- PDF 업로드 성공 시 드롭다운을 `pdf_complete`로 변경 (로컬)
- 서버 반영은 완료 버튼으로

### 3.3 미저장 상태 시각화 (0.3.21 → 0.3.22)

각 행에 `<input type="hidden" class="pdf-saved-status">` 추가 (서버 저장 상태 기록).  
드롭다운 변경 시 hidden 값과 비교해 달라지면 행 배경을 노란색(`#fff8e1`)으로 표시.  
완료 버튼 저장 성공 시 hidden 값 업데이트 + 노란색 제거.

**CSS 버그 수정 (0.3.22)**: Bootstrap `.table`이 `<td>`에 `background-color: var(--bs-table-bg)`를 적용해 `<tr>`에 설정한 배경색을 덮어쓰는 문제 → `.row-unsaved > td { background-color: #fff8e1 !important; }` 클래스 방식으로 변경.

---

## 4. 논문 변경 이력 상세 CSS 수정 (`/kprdb/reference_history_detail/`) (0.3.23)

**원인**: 바깥 `.geo-detail-table` 안에 중첩 테이블로 diff 표시 → CSS `.geo-detail-table th { width: 180px }`가 안쪽 `<th width="100">`에도 적용 → field명 컬럼이 100px → 180px로 확대, 총 열 너비가 바깥 500px 컨테이너 초과 → "변경 후" 컬럼이 화면 밖으로 밀려 안 보임.

**수정**: 템플릿 전면 재작성.
- 바깥 테이블에서 width/align 인라인 속성 제거, Bootstrap `<thead>` 적용
- 안쪽 diff 테이블은 `.geo-detail-table` 대신 독립 `.diff-table` 클래스 사용
- 퍼센트 기반 열 너비 (`38% / 38%`)로 "변경 전"/"변경 후" 컬럼이 항상 가시 범위 안에 오도록
- `word-break: break-word`로 긴 텍스트(abstract 등) 줄바꿈 처리

---

## 변경 파일

- `deploy/build.sh` — 빌드 자동화 스크립트 (신규)
- `fsis/views.py` — `index()` 비로그인 사용자 처리
- `kprdb/tests.py` — `test_index_anonymous` 회귀 테스트
- `kprdb/templates/kprdb/my_pdf_uploads.html` — 완료 버튼, 행 제거, 미저장 시각화
- `kprdb/templates/kprdb/reference_history_detail.html` — diff 테이블 재작성

## 커밋

- `b32c60f` — Fix 500 error on dashboard for anonymous users
- `47620c8` — Add regression test for anonymous user 500 on dashboard
- `5d28d07` — Add build.sh: test → version bump → docker build/push
- `54fa5b0` — Hide row on status change when new status doesn't match current filter
- `02ecc96` — Replace auto-save status with explicit 완료 button on PDF upload page
- `f8bf318` — Highlight unsaved status changes in PDF upload rows
- `f4656e3` — Fix row highlight not showing due to Bootstrap table CSS override
- `5ded9c2` — Fix history detail: 'after' column hidden due to nested table width overflow
