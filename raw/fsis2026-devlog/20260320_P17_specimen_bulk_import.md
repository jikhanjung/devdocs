# P17: 표본 일괄 입력 기능 구현

**작성일**: 2026-03-20
**상태**: 구현 완료 (2026-03-20)

---

## 배경

표본 데이터를 한 건씩 입력 폼으로 등록하는 방식은 대량 데이터 입력 시 비효율적이다.
Excel 파일로 다수의 표본을 일괄 입력할 수 있는 기능과 표준 입력 양식(템플릿)이 필요하다.

- 대상: fsis 앱의 MuseumSpecimen (화석 표본, `body_or_trace` 기준 체화석/생흔화석 구분)
- ghdb의 암석 표본은 별도 계획에서 처리

---

## 목표

1. **Excel 템플릿 제공** — 체화석용 / 생흔화석용 두 종류
2. **일반 사용자 업로드 페이지** — Admin 아닌 일반 로그인 사용자가 사용
3. **미리보기·검증** — 업로드 직후 행별 오류/경고를 확인하고 확정 저장
4. **fossil_site 연결** — site_code 텍스트로 FossilSite FK 매칭, 없으면 경고 후 미연결 저장
5. **저장 후 편집 가능** — 기존 edit_museum_specimen 뷰 그대로 활용

---

## 전체 흐름

```
① 업로드 페이지 접속
   /fsis/import_specimens/
   - 체화석 템플릿 다운로드
   - 생흔화석 템플릿 다운로드
   - xlsx 파일 업로드 폼

② 파싱 & 검증 (서버)
   - openpyxl로 3행부터 읽기 (1행: 한글헤더, 2행: 영문필드명)
   - 행별 유효성 검사 (필수값, choices, 날짜 형식, site_code 매칭)
   - 결과를 session에 저장

③ 미리보기 페이지
   /fsis/import_specimens/preview/
   - 녹색: 정상 / 빨간: 오류 / 노란: 경고 행 표시
   - 요약: "총 N행 — 정상 X행, 오류 Y행, 경고 Z행"
   - "오류 행 제외하고 저장" / "취소" 버튼

④ 확정 저장
   - 유효 행만 MuseumSpecimen.objects.bulk_create()
   - created_by = 현재 사용자
   - body_or_trace = 템플릿 종류에서 자동 설정
   - session 삭제 → museum_specimen_list로 리다이렉트
```

---

## Excel 템플릿 컬럼

### 공통 (체화석 / 생흔화석 동일)

| 열 | 한글 헤더 | 필드명 | 필수 | 비고 |
|----|-----------|--------|:----:|------|
| A | 표본번호 | specimen_code | ✓ | |
| B | 국가등록번호 | national_registration_code | | |
| C | 기관표본번호2 | specimen_code2 | | |
| D | 일반명 | common_name | | 드롭다운 |
| ... | (분류 필드 — 아래 참고) | | | |
| 지질시대 | geological_age | | | |
| 지질시대코드 | geological_age_code | | 드롭다운 |
| 분지명 | basin_name | | | |
| 지층명 | formation_name | | | |
| 암상 | lithology | | | |
| 산출지 | discovery_location | | | |
| GPS | gps_data | | | DMS 또는 십진도 |
| 표본크기 | specimen_size | | | |
| 보존부위 | preserved_part | | | |
| 보존상태 | preservation_status | | 드롭다운 |
| 반대면유무 | has_counterpart | | 드롭다운 |
| 수장기관 | institute_name | | | |
| 취득유형 | acquisition_type | | 드롭다운 |
| 제공자 | provider | | | |
| 취득일 | acquisition_date | | YYYY-MM-DD |
| 참고문헌1 | reference_info | | | |
| 화석산지코드 | fossil_site | | site_code 텍스트 |
| 입력자 | entry_workers | | | |
| 입력일 | entry_date | | YYYY-MM-DD |
| 비고 | entry_comments | | | |

### 체화석 전용 (D열 이후 삽입)

| 한글 헤더 | 필드명 | 비고 |
|-----------|--------|------|
| 분류체계 | general_classification | 드롭다운 |
| 계 | kingdom_name | |
| 문 | phylum_name | |
| 강 | class_name | |
| 목 | order_name | |
| 과 | family_name | |
| 속 | genus_name | |
| 종 | species_name | |
| 학명 | scientific_name | |
| 모식종 | type_species | |
| 모식표본구분 | is_type | 드롭다운 |

### 생흔화석 전용 (D열 이후 삽입)

| 한글 헤더 | 필드명 | 비고 |
|-----------|--------|------|
| 행동유형 | trace_behavior | 드롭다운 |
| 생흔화석과 | trace_family | |
| 생흔화석속 | trace_genus | |
| 생성동물 | trace_agent | 드롭다운 |

### 템플릿 구조

- **Sheet1 (데이터 입력)**: 1행 한글헤더(굵게, 배경색), 2행 영문 필드명(회색), 3행 예시 데이터
- **Sheet2 (유효값)**: choices 필드별 허용값 목록 (드롭다운 Data Validation의 source)

---

## 구현 파일

```
fsis/
├── import_utils.py               (신규) Excel 파싱·검증·템플릿 생성
├── views_import.py               (신규) import 관련 뷰 3개
├── urls.py                       (수정) URL 3개 추가
└── templates/fsis/
    ├── import_specimens.html          (신규) 업로드 페이지
    └── import_specimens_preview.html  (신규) 미리보기 페이지
```

---

## URL

```python
path('import_specimens/', views_import.import_specimens, name='import_specimens'),
path('import_specimens/preview/', views_import.import_specimens_preview, name='import_specimens_preview'),
path('download_specimen_template/<str:template_type>/', views_import.download_specimen_template, name='download_specimen_template'),
# template_type: 'body' (체화석) | 'trace' (생흔화석)
```

---

## 검증 규칙

| 조건 | 결과 |
|------|------|
| specimen_code 비어있음 | 오류 — 저장 불가 |
| choices 필드 허용값 외 입력 | 오류 — 저장 불가 |
| 날짜 형식 오류 (acquisition_date 등) | 오류 — 저장 불가 |
| fossil_site 코드가 DB에 없음 | 경고 — fossil_site=None으로 저장 |
| 빈 행 | 무시 (스킵) |

---

## 구현 순서

1. `import_utils.py` — `generate_template()`, `parse_excel()`, `validate_row()`, `save_rows()`
2. `views_import.py` — `download_specimen_template`, `import_specimens`, `import_specimens_preview`
3. `fsis/urls.py` — URL 3개 추가
4. `import_specimens.html` — 업로드 페이지
5. `import_specimens_preview.html` — 미리보기 테이블
6. `museum_specimen_list.html` — "일괄 입력" 버튼 추가

---

## 의존성

- `openpyxl` — Excel 생성·파싱 (requirements.txt 추가 필요 여부 확인)
- `django-import-export` 4.4.0 — 이미 설치, 이번 구현에서는 직접 openpyxl 사용 (미리보기 UX 제어를 위해)

---

## 완료 기준

- [x] 계획 문서 작성
- [x] 체화석·생흔화석 템플릿 다운로드 가능
- [x] xlsx 업로드 → 미리보기 (정상/오류/경고 행 구분)
- [x] 확정 저장 → museum_specimen_list 이동 (저장 건수 알림 표시)
- [x] 저장된 표본을 edit_museum_specimen에서 편집 가능 (기존 뷰 활용)
- [x] openpyxl==3.1.5 의존성 requirements.txt 반영
