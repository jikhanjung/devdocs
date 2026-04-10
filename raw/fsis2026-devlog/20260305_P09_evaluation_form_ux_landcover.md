# P09: 평가 폼 UX 개선 및 토지피복 자동분석

**작업일**: 2026-03-05
**커밋**: 0f2ba25

## 변경 요약

### 1. 스크린샷 폼 UI 개선

- **줌레벨 입력** 너비 70px로 축소 (기존: 기본 너비로 인해 줄넘김 발생)
- **Wayback 날짜** `type="date"` 위젯 적용 (달력 선택)
- **지도유형 choices** 한글 통일: `Hybrid` → `하이브리드`, `Roadmap` → `도로`, `Skyview` → `위성`
- **레이블 통일**: 동적 추가 폼의 `Image` → `이미지`

### 2. 스크린샷 폼 동적 추가/삭제

- **"스크린샷 폼 추가" 버튼**: Django formset management form의 TOTAL_FORMS 증가시키며 새 행 추가
- **빈 폼 삭제**: 우상단 × 버튼으로 미저장 폼 행 즉시 제거
- **기존 저장 스크린샷**: DELETE 체크박스로 삭제 (Django inline formset 기본 동작)
- 각 행을 카드형 `screenshot-row`로 감싸 시각적 구분

### 3. 스크린샷 라이트박스 (팝업 확대)

- 기존 저장 스크린샷: 120×90 썸네일 클릭 시 전체화면 오버레이로 확대
- 캡처한 스크린샷: 160×120 썸네일 클릭 시 동일 라이트박스
- 배경 클릭으로 닫기

### 4. 토지피복 자동분석 (서버 API)

gheval 데스크톱 앱의 `GhLandCover.py` 알고리즘을 서버 API로 포팅.

**알고리즘**: HSV 색공간 + Excess Green Index (ExG) 기반 픽셀 분류
- 밀집식생: H:35-85, S>40%, V>30%, ExG>0.05
- 희소식생: H:35-85, S:20-40% 또는 ExG:0~0.05
- 나지: H:10-30, S:20-60%
- 시가지: S<20% (회색 톤)
- 수계: H:90-130, S>30%, V<60%

**구현**:
- `evaluator/services/landcover.py` — `analyze_landcover_from_bytes()` 추가 (PIL/NumPy, OpenCV 불필요)
- `evaluator/views.py` — `analyze_landcover_api` 엔드포인트 (POST)
- `evaluator/urls.py` — `/evaluator/api/analyze-landcover/<site_pk>/`

**프론트엔드 흐름**:
1. "자동분석" 버튼 클릭
2. 위성 뷰 전환 + 줌 15 + 분석원 표시
3. 2.5초 대기 (타일 로딩)
4. `map.getCanvas().toDataURL()` 캡처
5. 서버 POST → 결과를 폼 필드에 자동 입력

## 변경 파일

| 파일 | 변경 |
|------|------|
| `evaluator/forms.py` | 스크린샷 폼 위젯 (줌레벨 너비, 날짜 picker) |
| `evaluator/models.py` | MAP_TYPE_CHOICES 한글화 |
| `evaluator/services/landcover.py` | `analyze_landcover_from_bytes()` 추가 |
| `evaluator/views.py` | `analyze_landcover_api` 뷰 추가 |
| `evaluator/urls.py` | 토지피복 분석 API URL 추가 |
| `evaluator/templates/evaluator/evaluation_form.html` | 동적 폼, 라이트박스, 자동분석 UI |
