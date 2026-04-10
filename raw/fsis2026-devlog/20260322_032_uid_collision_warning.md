# 032. UID 충돌 경고 기능 추가

**날짜**: 2026-03-22

## 배경

UID 생성 시 동일한 UID가 이미 존재하면 조용히 `-c2` 접미사를 붙여 저장하고 있었음.
같은 UID가 나온다는 것은 중복 문헌일 가능성이 높으므로, 사용자에게 경고를 주어야 함.

## 변경 내용

### 1. 개별 UID 생성 (`kprdb/views.py` — `_process_generate_uid`)
- 충돌 시 `warning` 필드에 기존 문헌 ID와 제목을 포함하여 JSON 응답 반환
- 정상 생성 시에는 warning 없음

### 2. 일괄 UID 생성 (`kprdb/management/commands/generate_uids.py`)
- 충돌 발생 시 콘솔에 WARNING 레벨로 충돌 문헌 쌍 정보 출력
- 예: `UID 충돌 [123] vs [ID 45: Kim et al. 2020] — 중복 문헌 확인 필요 → ...`

### 3. 프론트엔드 (`kprdb/templates/kprdb/reference_detail.html`)
- 개별 실행: UID 충돌 시 badge가 노란색(warning)으로 표시 + 로그에 경고
- 전체 실행(run_all): 각 step 결과에 warning 있으면 로그에 표시

## 기타
- HANDOFF.md: Project Structure에서 evaluator 중복 기재 수정
