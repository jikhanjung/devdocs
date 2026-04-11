# 작업 관리 시스템

교수가 학생에게 논문 처리 작업을 할당하고, 진행 상황을 추적하는 시스템.

## TaskAssignment 모델

- 6단계 상태 워크플로우
- 3단계 작업 Phase:
  - **Phase 1** (PHASE_PDF=1): PDF 확보 — finding_pdf → complete/unavailable
  - **Phase 2** (PHASE_ENTRY=2): 데이터 입력
  - **Phase 3** (PHASE_REVIEW=3): 검토

### Phase 1 상태 변천
초기에 pending_pdf 상태 존재 → 제거하고 finding_pdf로 시작 (182건 전환)

## 작업 할당 UI

- 2패널 레이아웃: 논문 선택 + 할당 상세
- AJAX 논문 목록 로딩, "no PDF" 필터
- 작업자별 배분: 81건(관리자 컨퍼런스 초록), 68건(kchwoo9 jpaleodb), 30건(10명 분배)
- 삭제 버튼 + 확인 다이얼로그

## 작업 워크스페이스 (task_workspace)

왼쪽-오른쪽 분할 레이아웃:

### 왼쪽 패널: PDF 뷰어
- PDF 없는 경우 드롭존 표시
- click-drag 패닝 (cursor: grab/grabbing)
- zoom-to-width 초기 표시
- OCR 텍스트 탭

### 오른쪽 패널: 데이터 입력
- 스크롤 가능한 상/하 분할
- **Site Candidates** 카드: 화석산지 후보
- **Taxa** 카드: 분류군 (산지별 border 카드 그룹핑)
- **Specimens** 카드: 표본
- AJAX API (15개 엔드포인트)
- 상태 자동 전환
- page_no 자동 기록 (PDF 뷰어에서)
- type_specimen 필드 (holotype/paratype/syntype/lectotype/neotype) + 배지
- commonname 필드 드롭다운
- fossil site remarks 필드
- task notes: debounce 자동저장

## my_tasks 페이지

- PDF 아이콘 컬럼, 제목 컬럼
- PDF 확보 상태 필터 드롭다운
- table-layout:fixed + colgroup 폭 제약 (author 150px, title auto, status 80px, remarks 200px)
- PDF 섹션 collapse/expand (chevron 애니메이션)

## my_pdf_uploads 페이지

- 드래그앤드롭 PDF 업로드 + Google Scholar 검색 링크
- 상태 변경 시 row fade-out
- 미저장 상태 노란색 하이라이트 (#fff8e1)
- 컬럼 순서: author/title/journal/page/search/status/PDF

## pdf_unavailable_list

- 관리자 전용: PDF 확보 불가 논문 목록
- navbar 메뉴 추가

## 대시보드 통합

- 작업 통계: PDF 확보/데이터 입력/검토/완료 건수
- Phase별 색상 바 (green/red/blue)
- "내 작업" 카드
- AI 추출 건수 표시 (today 배지)
- 최근 활동 피드

## 권한 제어

- **Undergraduate Students** 그룹: 보기 전용 (can_edit=False)
- 30개 뷰에 403 Forbidden 체크
- 26개 템플릿에서 편집/삭제/추가 버튼 숨김
- 6개 CBV에 get_context_data 오버라이드

## 관련 페이지

- [PDF 파이프라인](pdf-pipeline.md) — PDF 뷰어 및 추출 통합
- [KPRDB](kprdb.md) — Reference 모델
- [프로젝트 개요](fsis2026-overview.md)

---
*Sources: 034, 035, 036, 045, 051, 057, 058, 059, 060, 063, 065, P18, R01*
