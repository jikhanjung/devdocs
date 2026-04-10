# 065 — 대시보드 작업현황 개선, PDF 확보 불가 목록, 작업 UI 개선

**날짜**: 2026-04-10
**버전**: 0.3.53~0.3.61 (미커밋 변경 포함)

## 작업 내용

### 1. 대시보드 AI추출 건수 표시
- 논문 카드에 "AI추출 N건" 추가 (`.extract.claude.json` 파일 존재 확인)
- 오늘 추출된 건수도 today 배지로 표시
- `fsis/views.py`: PDF 있는 논문의 `.extract.claude.json` 파일 존재 여부를 순회하며 카운트

### 2. 대시보드 작업현황 단계별 분리
- 기존: 전체/대기/진행중/완료 4개 숫자 + 단일 progress bar
- 변경: PDF 확보 / 정보 입력 / 정보 검토 3단계로 분리
  - PDF 확보: 진행/완료/불가 + progress bar (green/red/blue)
  - 정보 입력: 진행/완료 + progress bar
  - 정보 검토: 진행/완료 + progress bar
- `fsis/views.py`: `task_stats`를 phase별 쿼리로 변경
- `fsis/templates/fsis/dashboard.html`: 레이아웃 전면 재구성

### 3. PDF 확보 불가 목록 페이지 (신규)
- `kprdb/views_task.py`: `pdf_unavailable_list` 뷰 추가 (관리자 전용)
- `kprdb/templates/kprdb/pdf_unavailable_list.html`: 작업자/저자/제목/저널/비고 테이블
- `kprdb/urls.py`: `pdf_unavailable/` 경로 추가
- 작업자별, 연도순 정렬

### 4. Navbar에 PDF 확보 불가 메뉴 추가
- `fsisnavbar.html`: 작업관리 드롭다운에 "PDF 확보 불가" 항목 추가

### 5. 작업할당 목록 UI 개선
- `task_assignment_list.html`: 비고 컬럼 max-width 250px + word-break + urlize
- `task_worker_detail.html`:
  - PDF 유무 아이콘 컬럼 추가 (초록 PDF 아이콘 / 회색 대시)
  - 비고 컬럼 urlize + max-width
  - 삭제 버튼 추가 (확인 dialog)
  - colspan 6 → 7 수정
