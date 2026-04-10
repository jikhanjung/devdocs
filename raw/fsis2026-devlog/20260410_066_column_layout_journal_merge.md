# 066 — 테이블 컬럼 폭 고정, 저널 병합, 저널명 언어별 표시

**날짜**: 2026-04-10

## 작업 내용

### 1. 내 작업 페이지 컬럼 폭 고정 (`my_tasks.html`)
- `table-layout:fixed` + `colgroup`으로 컬럼 폭 고정
- PDF 확보 테이블: 저자 150px, 제목 auto, 상태 80px, 비고 200px
- 자료 입력/검토 테이블: 저자 150px, 제목 auto, 단계 80px, 상태 80px, 할당일 90px, 비고 200px
- 긴 텍스트 `text-truncate`, 비고 `urlize` + `word-break:break-all`

### 2. 논문 목록 페이지 컬럼 개편 (`reference_list.html`)
- 저자 + 연도 컬럼 → "저자 (연도)" 단일 컬럼으로 병합 (연도 컬럼 삭제)
- `table-layout:fixed` + `colgroup`: 유형 30px, 저자 160px, 제목 auto, PDF 45px, Taxa 55px, Specimens 75px
- colspan 7 → 6

### 3. 저널 데이터 정리 (프로덕션 DB)
- 저널 425 "고생물학회지" → 423 "한국고생물학회지"로 논문 86건 이동
- 저널 423 → 422 "Journal of the Paleontological Society of Korea"로 논문 108건 이동
- 422번 저널에 `title_k = '한국고생물학회지'` 설정 (영문+한글 통합)
- 결과: 422번 저널 413건 (기존 305 + 108)

### 4. 논문 상세 저널명 언어별 표시 (`reference_detail_body.html`)
- `reference.language == 'EN'` → 저널 영문제목 우선 (없으면 원어제목 fallback)
- 그 외 → 저널 원어제목 우선 (없으면 영문제목 fallback)
- 기존: `journal.get_title()` (항상 원어제목 우선)
