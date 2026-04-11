# 068 — 저널 데이터 정리, 학명 이탤릭, 저널 목록 논문 수 표시

**날짜**: 2026-04-11

## 작업 내용

### 1. 저널 데이터 정리 — 중복 병합 (프로덕션 DB)
- IGCP-350 저널 5개 → 628번으로 통합 (6건)
  - 625, 629, 630, 631: 오타/대소문자 차이로 분리되어 있던 동일 저널
- 459 "Geosciences Journa"(오타) → 451 "Geosciences Journal" (9건)
- 563 "ICHNOS-AN INTERNATIONAL JOURNAL..." → 435 "Ichnos" (4건)
- 430 "Journal of paleontology"(소문자) → 428 "Journal of Paleontology" (2건)
- 427 "지구과학회지" + 431 "Journal of Korean Earth Science Society" + 513 → 470 (총 68건)
  - 470에 영문/원어 제목 모두 설정
- 426 "Journal of Geological Society of Korea" → 429 "지질학회지" → 420 (총 236건)
  - 420에 영문/원어 제목 모두 설정 (Journal of the Geological Society of Korea / 지질학회지)
- 병합된 빈 저널 모두 `is_deleted=True` soft delete

### 1b. 논문별 저널 수동 변경 (프로덕션 DB, 웹 UI)
- ref 2102, 2103 (Kim and Kimura 1987): 424 Proceedings of the Japan Academy Series B → 606 Proceedings of the Japan Academy, Series B
- ref 2155 (Lee 1994): 438 Paleontology → 481 Palaeontology
- ref 2331 (Park and Choi 2011): 479 Geological magazine → 462 Geological Magazine
- ref 2520 (Yi and Yun 1995): 507 Paleontographica Abteilung B → 607 Palaeontographica, Abteilung B
- ref 2672, 2679, 2813: 531 한국지구과학회 → 552 한국지구과학회 학술발표회 논문집
- ref 2710, 3610: 532 Alcheringa: An Australasian Journal of Palaeontology → 458 Alcheringa
- ref 2936 (이연규와 윤선 2004): 493 한국고생물학회 → 495 한국고생물학회 창립 20주년 기념 "한국 고생물"
- ref 3571 (강진건 2019): 642 Journal of Kim Il Sung University → 640 Journal of Kim Il Sung University (Earth Science and Geology)
- ref 2372, 2376, 2387 (최덕근): 492/497 출판사 → journal=None (도서, 저널 아님)
- 이동 후 빈 저널 soft delete: 424, 438, 479, 507, 531, 532, 493, 497, 492, 485, 477, 482, 508, 566, 642

### 2. 학명 이탤릭 표시
- `scientific_name` 필드에 저장된 `<i>...</i>` 태그가 이탤릭으로 렌더링되도록 `|safe` 필터 추가
- 적용 템플릿 5개:
  - `evaluator/type_specimen_list.html`
  - `evaluator/specimen_assessment_detail.html`
  - `evaluator/specimen_assessment_form.html`
  - `evaluator/site_specimen_list.html`
  - `fsis/fossil_site_detail.html`

### 3. 저널 목록 논문 수 표시
- `journal_list` 뷰: `annotate(ref_count=Count('reference'))` 추가
- `journal_list.html`: "논문" 컬럼 추가 (60px, 중앙 정렬)
