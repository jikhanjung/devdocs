# UI 디자인 시스템

FSIS2026의 프론트엔드 테마 및 UI 컴포넌트 체계.

## Earth-tone 테마

86개 파일에 걸친 전면 UI 리뉴얼 (2026-03-05).

### CSS 변수 체계
- **geo-theme.css**: 공유 CSS 파일
- FSIS 팔레트: Earthy/Amber (#b8860b)
- KPRDB 팔레트: #3d6b5e
- Bootstrap 5.3 기반
- Google Fonts: Noto Sans KR

### 컴포넌트 클래스
- `.geo-card`, `.geo-page-header`, `.geo-table`
- data-label 속성: 모바일 반응형 테이블 fallback
- 반응형 breakpoints: 768px, 576px

## 아이콘 체계

Bootstrap Icons 전면 채택:
- 문헌유형: book (BK/BS), file-text (JN/RP), mortarboard (TH)
- PDF 상태: bi-file-earmark-pdf
- 작업: grip 아이콘 (드래그앤드롭), chevron (collapse)

## 네비게이션

- FSIS + KPRDB 통합 navbar (fsisnavbar.html)
- 드롭다운: 논문정보, 기본정보, 시스템 관리
- 교수/일반 사용자 별도 메뉴
- KOFHIN 브랜딩
- 프로필 링크, 로그인 상태 표시

## 폼 디자인

- Bootstrap5 수평 레이아웃
- 문헌유형별 동적 필드 표시/숨김
- 드래그앤드롭 저자 순서 변경
- 커스텀 autocomplete + 신규 생성 모달
- debounce 자동저장 (task notes)
- 평가 폼: 동적 행 추가/삭제, formset 관리

## 테이블 레이아웃

- table-layout:fixed + colgroup 폭 제약
- author+year 컬럼 병합
- 압축 헤더: Taxa→Tx, Specimens→Sp
- title 툴팁
- text-truncate, urlize 포맷팅
- 정렬 화살표 표시

## 대시보드

- 통계 카드: 표본/논문/화석산지/분류군/평가 건수
- 작업 진행 카드 (Phase별 색상 바)
- AI 추출 건수 (today 배지)
- 최근 활동 피드
- 익명 사용자 500 에러 수정 (AnonymousUser 체크)

## Lightbox

- 순수 JS/CSS 구현 (외부 라이브러리 없음)
- 이전/다음 네비게이션 (화살표 버튼 + 키보드)
- HTML5 dataset 속성
- 표본 사진 갤러리, 평가 이미지에 적용

## 기타 UX 개선

- PDF 업로드: row fade-out, 미저장 노란색 하이라이트
- PDF 뷰어: click-drag 패닝, cursor: grab/grabbing
- PDF 섹션: collapse/expand + chevron 애니메이션
- 저널명: 언어별 표시 (영어 논문 → title_e 우선)
- 저널 병합: 3건 → 1건 통합 (한국고생물학회지, 413건)

## 관련 페이지

- [프로젝트 개요](fsis2026-overview.md)
- [작업 관리](task-management.md) — 작업 UI 상세
- [KPRDB](kprdb.md) — 논문 목록 UI

---
*Sources: 003, 006, 009, 012, 013, 014, 015, 033, 036, 043, 051, 053, 056, 060, 063, 064, 065, 066, 067, P04*
