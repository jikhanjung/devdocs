# 063 — UI 개선 및 PDF UX (v0.3.31~0.3.38)

**날짜**: 2026-04-08  
**버전**: 0.3.31 → 0.3.38

## 작업 내용

### 1. Claude 보강 cron 간격 변경 (v0.3.31)
- `scripts/claude_augment.py` 기본 모델 `opus` → `sonnet`으로 변경
- cron 간격 10분 → 30분으로 변경 (비용 절감)
- `/srv/fsis2026/scripts/claude_augment.py`에 직접 복사 적용
- CLAUDE.md, HANDOFF.md, README.md 문서 업데이트

### 2. my_tasks PDF 확보 필터 추가 (v0.3.31)
- `kprdb/views_task.py` `my_tasks` 뷰에 `filter_status` 쿼리파라미터 처리 추가
- `my_tasks.html` PDF 확보 섹션에 상태 필터 드롭다운 추가 (기본: 전체)

### 3. my_pdf_uploads 레이아웃 개선 (v0.3.32~0.3.34)
- 컬럼 순서 재편: 저자(연도) / 제목 / 저널 / 페이지 / 검색 / 상태·메모 / PDF
- `short_title|default:author_list` 형식으로 저자(연도) 컬럼 통일
- 저자(연도), 제목 양쪽 모두에 `reference_detail` 링크 추가
- 페이지 컬럼 복원 (v0.3.34에서 재추가)
- **500 에러 수정**: `journal_title|default:a.reference.journal.get_title` 패턴이 Django 5.x에서
  None FK의 `@property` 접근 시 AttributeError를 재발생시키는 문제 → `{{ a.reference.journal_title|default:"" }}`로 단순화

### 4. my_tasks 링크 및 레이아웃 개선 (v0.3.35)
- PDF 확보 테이블: 저자(연도), 제목 양쪽에 `reference_detail` 링크 추가
- 자료 입력/검토 테이블: 저자(연도), 제목 양쪽에 `task_workspace` 링크 추가

### 5. task_workspace PDF 뷰어 개선 (v0.3.35~0.3.37)
- **클릭-드래그 패닝**: `mousedown`/`mousemove`/`mouseup` 이벤트로 `scrollLeft`/`scrollTop` 조작
  - `.pdf-viewport`에 `cursor:grab`, 드래그 중 `cursor:grabbing`
- **초기 줌 맞춤(width 100%)**: 페이지 로드 시 `setZoom(1)` 호출하여 뷰포트 너비에 맞게 표시
- **줌 방식 변경**: CSS `transform: scale()` → `img.style.width` 퍼센트 방식
  - transform은 레이아웃에 영향 없어 overflow scroll 불가 → width 방식으로 실제 레이아웃 확장

### 6. my_tasks PDF 확보 섹션 접기/펼치기 (v0.3.36)
- Bootstrap collapse 적용 (`#pdfSection`)
- 헤딩의 텍스트/아이콘 `<span>`에만 `data-bs-toggle` 적용 (버튼과 분리)
  - 전체 `<h5>`에 적용 시 자식 버튼 클릭도 collapse 트리거되는 문제 방지
- 접기/펼치기 시 chevron 아이콘 회전 애니메이션

### 7. pdf_status_by_year 헤더 수정 (v0.3.38)
- "통합 파이프라인" → "AI 추출"로 컬럼 헤더 변경

## 주요 교훈

- Django 5.x에서 `|default:` 필터의 인자로 None FK의 dotted path 사용 시 500 에러 발생 가능
  - `None` FK에 대한 property 접근이 AttributeError를 re-raise함
  - 해결: DB 필드(denormalized) 직접 참조하거나 뷰에서 미리 처리
- Bootstrap collapse는 `data-bs-toggle` 지정 요소의 모든 자손 클릭을 캡처함
  - 별도 액션 버튼이 있으면 toggle 대상을 별도 `<span>`으로 분리할 것
- CSS `transform: scale()`은 레이아웃 크기 변경 없이 시각적 변환만 → overflow scroll 불작동
  - 스크롤 필요 시 `width` 퍼센트 조작이 적합
