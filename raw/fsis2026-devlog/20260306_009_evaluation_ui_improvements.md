# 009: 위험성 평가 UI 개선

**날짜**: 2026-03-05 ~ 2026-03-06

## 관련 계획
- `20260305_P09_evaluation_form_ux_landcover.md`

## 작업 내용

### 1. 평가 폼 UX 개선 (2026-03-05)
- 스크린샷 폼: 동적 행 추가/삭제 기능
- 이미지 라이트박스 팝업
- 날짜 선택 위젯 (date picker)
- 라벨 통일 (이미지, 하이브리드/도로/위성)

### 2. 토지피복 분석 API 추가 (2026-03-05)
- HSV+ExG 기반 위성 이미지 토지피복 분류
- Auto-analyze 버튼: 위성뷰 전환 → 캔버스 캡처 → 서버 분석
- `evaluator/services/landcover.py` 신규 생성

### 3. EvaluationScreenshot → EvaluationImage 리네임 (2026-03-06)
- RenameModel 마이그레이션 (데이터 보존)
- 이미지 분류 필드 추가: 스크린샷 / 현장사진 / 기타
- 설명 필드 추가

### 4. 평가 폼 추가 개선 (2026-03-06)
- 스크린샷 캡처 시 4:3 비율 (1200x900) 적용
- Brilha 5개 항목에 점수 방향 설명 추가 (1=낮음 → 5=높음)
- 새 평가 시 평가일 오늘 날짜 자동 설정
- 산지 위경도 표시
- 이미지 섹션 전체 너비 레이아웃

## 수정 파일
- `evaluator/models.py` — EvaluationImage 모델, 분류/설명 필드
- `evaluator/forms.py` — 폼 필드 추가
- `evaluator/views.py` — 이미지 처리 뷰 수정
- `evaluator/admin.py` — admin 등록 수정
- `evaluator/templates/evaluator/evaluation_form.html` — 폼 UI 전면 개선
- `evaluator/templates/evaluator/evaluation_detail.html` — 상세 보기 수정
- `evaluator/services/landcover.py` — 토지피복 분석 서비스
- `evaluator/urls.py` — 분석 API URL 추가
