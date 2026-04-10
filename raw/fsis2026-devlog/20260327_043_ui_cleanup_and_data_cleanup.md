# 043. UI 개선 및 데이터 정리

**날짜**: 2026-03-27
**버전**: 0.2.80 → 0.2.81

## 1. 논문 목록 필터 개선 (v0.2.80)

- "PDF 없는 논문" 필터 옵션 추가
- 드롭다운을 검색어 왼쪽으로 이동
- 드롭다운 화살표 표시: geo-theme.css에서 select와 input 스타일 분리, SVG 화살표 배경이미지 적용

## 2. 저자 편집 폼 개선 (v0.2.81)

- `kprdb/templates/kprdb/author_form.html`: `{{ form }}` 일괄 렌더링 → 개별 필드 테이블 레이아웃
- 라벨: 성(한글), 이름(한글), 성(영문), 이름(영문), 소속, Is primary, Redirect to, 비고
- 입력 필드 통일 스타일 적용

## 3. 북한 논문 저자 한글 이름 매칭

- 영문 이니셜 → 한글 이름 매칭: **38명** 업데이트
  - 확인된 매칭 34명, 추정 매칭 4명
  - 미매칭 13명 (중국/일본 연구자 및 확인 불가)
- 이니셜 불일치 사례: 장덕성(Jang T.S. / Jang D.S.), 리상도(Li S.D. / Ri S.D.)
- 상세: `docs/north_korea_author_name_mapping.md`

## 4. 데이터 정리

- Geosciences Journal 한국 무관 논문 **32건 삭제** (인도, 말레이시아, 남극 등)
- PDF 미보유 20건 목록: `docs/geosciences_journal_no_pdf.txt`

## 5. DB 현황

| 구분 | 건수 |
|------|------|
| 전체 논문 | 1,383건 |
| PDF 보유 | 834건 (60%) |

## 수정 파일

| 파일 | 작업 |
|------|------|
| `kprdb/views.py` | "PDF 없는 논문" 필터 추가 |
| `kprdb/templates/kprdb/reference_list.html` | 드롭다운 위치 변경, 인라인 스타일 제거 |
| `static/geo-theme.css` | select 화살표 스타일 |
| `kprdb/templates/kprdb/author_form.html` | 테이블 레이아웃으로 변경 |
| `docs/north_korea_author_name_mapping.md` | **신규** — 한글 이름 매칭 문서 |
| `docs/geosciences_journal_no_pdf.txt` | **신규** — GJ PDF 미보유 목록 |
