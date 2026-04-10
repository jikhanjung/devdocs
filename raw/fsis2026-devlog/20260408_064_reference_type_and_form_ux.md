# 064 — 문헌유형, 도서 필드, 편집 폼 UX 개선 (v0.3.39~0.3.52)

**날짜**: 2026-04-08
**버전**: 0.3.39 → 0.3.52

## 작업 내용

### 1. 문헌유형(type) UI 노출 (v0.3.39~0.3.40)
- 기존 `type` 필드(BK/JN/RP/TH)가 UI에 미노출 → 3곳에 추가
  - `reference_list.html`: 저자 앞 아이콘 컬럼 (JN 빈칸, BK=book, BS=bookmark, RP=file-text, TH=mortarboard)
  - `reference_detail_body.html`: "문헌유형" 행 추가
  - `reference_form2.html`: 문헌유형 드롭다운 + "모든 항목 보이기" 체크박스
- 유형별 필드 표시/숨김 (JS toggle):
  - JN/RP/TH: 저널/권/호수 표시, 도서 필드 숨김
  - BK/BS: 저널/권/호수 숨김, 도서 필드 표시
  - BS만: 책 제목 필드 표시

### 2. Book Section 타입 및 도서 전용 필드 추가 (v0.3.41)
- `REFERENCE_TYPE_CHOICES`에 `('BS', 'Book Section')` 추가
- 도서 필드 추가: `book_title`(책 제목), `publisher`(출판사), `place`(출판지), `series`(시리즈), `series_number`(시리즈 번호), `edition`(판), `isbn`(ISBN), `url`(URL)
- Zotero Book 항목 기준으로 필드 선정
- 마이그레이션: 0020, 0021, 0022

### 3. 편집 폼 필드 순서 재정렬 (v0.3.41)
- 분류 → 저자 → 제목 → 연도 → 언어 → 저널(JN) → 페이지 → 도서(BK/BS) → DOI/URL → 초록 → PDF → 화석종류 → 비고
- 도서 필드 위젯 폭 확대 (size=60, book_title은 Textarea 2행)

### 4. 저자 드래그 순서 변경 (v0.3.42~0.3.45)
- `author_order` 숫자 입력 → hidden 처리
- 드래그 핸들(grip-vertical 아이콘) 추가, HTML5 drag-and-drop으로 순서 변경
- 핸들에서만 드래그 시작 (mousedown → draggable 활성화)
- **버그 수정**:
  - `input[type="number"]` → `input` (실제는 type="text")
  - `select[name$="-author"]` → `input[type="hidden"][name$="-author"]` (Select2 → HiddenInput 변경 후)
  - `onsubmit` 글로벌 스코프 문제 → `addEventListener('submit')` 방식으로 변경
  - 저자 빈 행에는 `author_order` 비워두기 (formset 유효성 검사 실패 방지)

### 5. 저자 커스텀 autocomplete (v0.3.46~0.3.47)
- Select2 위젯 → 커스텀 텍스트 입력 + 드롭다운 목록
- 검색 API: `api/author_search/` (abbreviation_e/k icontains)
- 드롭다운 하단 "새 저자 추가" 버튼 → 모달 팝업 (성/이름 영문+한글)
- 생성 API: `api/author_create/` (generate_abbreviation 자동 호출)
- 한글/영문 자동 구분하여 prefill
- 폼 저장 실패 시 에러 메시지 표시 (상단 alert)

### 6. 저널 커스텀 autocomplete (v0.3.48)
- 저자와 동일한 방식 적용
- 검색 API: `api/journal_search/`, 생성 API: `api/journal_create/`
- 모달: 영문 제목 + 원어 제목

### 7. 저널 soft delete (v0.3.49)
- `Journal.is_deleted` 필드 추가 (BooleanField, default=False)
- `journal_delete` 뷰: `get_object_or_404(journal)` 오타 수정 (`Journal`), 논문 연결 시 삭제 차단 + 에러 메시지, 실제 삭제 대신 `is_deleted=True`
- journal_list, api_journal_search, JournalAutocomplete: `is_deleted=False` 필터 적용
- 마이그레이션: 0023

### 8. 기타 수정 (v0.3.50~0.3.52)
- `Reference.volume` max_length 5 → 30 (마이그레이션 0024)
- `reference_detail_body`: 저널명에 journal_detail 링크 추가
- `reference_form2`: 일반명 텍스트 입력 필드 숨김 (multiselect 체크박스만 표시)

### 9. OCR → Markdown 변환 스크립트
- `scripts/ocr_to_md.py`: `.pdf.json` → `.md` 변환 (페이지별 markdown 필드 이어붙이기)
- 로컬 30개, 프로덕션 1,198개 변환 완료 → `uploads/references/md/`
- MD 파일은 JSON 대비 약 27~40% 크기 (bbox/메타데이터 제거)

### 10. Claude 정보추출 cron 간격 변경
- claude_augment cron 간격 30분 → 20분으로 변경

## 주요 교훈
- Django formset의 `extra` 행에서 hidden 필드는 JS로 초기값을 채워줘야 유효성 검사 통과
- `onsubmit="fn()"` 속성은 글로벌 스코프에서 실행 → script 블록 내 함수는 `addEventListener` 사용
- SQLite는 `max_length`를 DB 레벨에서 강제하지 않음 (Django 폼에서만 검증)
- HTML5 drag-and-drop에서 `draggable` 속성을 핸들 mousedown에서만 활성화하면 다른 입력 요소와 충돌 방지 가능
