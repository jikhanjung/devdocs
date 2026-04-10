# 059. 대시보드 내 작업 섹션, Claude 추출 에러 처리 개선

**날짜**: 2026-04-05  
**버전**: 0.3.16 → 0.3.17

---

## 1. 대시보드 내 작업 섹션 추가

### 변경 내용

- `fsis/views.py`: `index()` 뷰에 `my_task_stats` 컨텍스트 추가
  - PDF 확보 중 / 자료 입력 중 / 검토 중 / 전체 / 완료 건수
  - `today_specimens/references/taxa`: `modified_on` → `created_on` 기준으로 수정 (오늘 수정이 아닌 오늘 생성된 항목 카운트)
- `fsis/templates/fsis/dashboard.html`:
  - 작업현황 col-md-6 → col-md-4 축소
  - "내 작업" 카드 (col-md-3) 신규 추가 — PDF확보/자료입력/검토/완료 건수 + "내 작업으로" 버튼
  - 최근 활동 col-md-6 → col-md-5 조정

---

## 2. Claude 추출 스크립트 에러 처리 개선 (`scripts/extract_from_ocr.py`)

### 2.1 Claude 사용량 초과 감지 및 중단

- `ClaudeUsageLimitError` 예외 클래스 추가
- `call_claude()`: stderr/stdout에서 `usage limit`, `rate limit`, `hit your limit` 등 키워드 감지 시 응답 메시지 출력 후 예외 발생
- `process_json()`: Claude augmentation 도중 사용량 초과 시
  - "saving regex-only result" 출력
  - summary까지 계산 완료 후 `ClaudeUsageLimitError(msg, result)` 발생 — regex 결과를 예외에 첨부
- 메인 루프: `ClaudeUsageLimitError` 수신 시 regex 결과 저장 → stats 업데이트 → 중단 메시지 출력 → `break`
- 사용량 초과 후 재실행 시 이미 저장된 파일은 `--force` 없이 자동 건너뜀

### 2.2 JSON 파싱 에러 처리

- **"Expecting value: char 0"** 에러: regex로 코드 펜스 제거 후 빈 문자열이 되는 경우 → 제거 후에도 `output.strip()` 체크 추가해 `None` 반환

- **"Extra data"** 에러: Claude가 JSON 뒤에 설명 텍스트를 추가 출력하는 경우
  - `json.loads()` 실패 시 `json.JSONDecoder().raw_decode()`로 재시도 — 첫 번째 완전한 JSON 객체만 추출

- **파싱 완전 실패 시**: 에러 메시지 + Claude 출력 첫 300자 표시 (원인 파악용)

---

## 변경 파일

- `fsis/views.py` — `my_task_stats`, `created_on` 기준 오늘 카운트
- `fsis/templates/fsis/dashboard.html` — 내 작업 카드
- `scripts/extract_from_ocr.py` — `ClaudeUsageLimitError`, JSON 파싱 개선
- `config/version.py` — 0.3.17

## 커밋

- `b35b1b8` — Add my_tasks section to dashboard, fix today count to use created_on
- `5de91f5` — Detect Claude usage limit and stop extraction pipeline early
- `3b83b11` — Save regex-only result before stopping on Claude usage limit
- `85d2af3` — Parse first JSON object only to handle extra text after Claude response
- `2444f0f` — Show Claude output on JSON parse failure for easier debugging
- `9223e3c` — Print Claude response message on usage limit detection
