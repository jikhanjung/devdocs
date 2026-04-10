# P19 — 논문 유형(type) UI 노출 및 활용

**날짜**: 2026-04-08

## 배경

Reference 모델에 서지학적 유형을 구분하는 `type` 필드가 이미 존재한다:
```python
REFERENCE_TYPE_CHOICES = [
    ('BK', 'Book'),
    ('JN', 'Journal Article'),
    ('RP', 'Report'),
    ('TH', 'Thesis'),
]
# default='JN'
```

그러나 이 필드가 UI 어디에도 노출되지 않아 사용자가 구분할 수 없는 상태다.
(`type2`는 분류학/비분류학/해외화석 구분 용도로 별개)

### 현재 데이터 현황 (dev DB 기준)
| type | 건수 | source |
|------|------|--------|
| BK (Book) | 13 | jpaleodb |
| JN (Journal Article) | 162 | jpaleodb |

- jpaleodb import만 `journal_title` 유무로 BK/JN 자동 분류
- dbpia, crossref 등 다른 import는 모두 기본값 `JN`으로 입력됨
- 수동 등록 논문도 기본값 `JN`

## 작업 계획

### 1단계: UI 노출 (완료)

#### 1-1. reference_list 테이블에 유형 아이콘 컬럼 추가 ✅
- 저자 컬럼 앞에 아이콘 표시 (JN은 빈칸, BK/RP/TH만 아이콘)
- 아이콘: `bi-book` (BK), `bi-file-text` (RP), `bi-mortarboard` (TH)
- 툴팁으로 전체 라벨 표시

#### 1-2. reference_detail에 문헌유형 행 추가 ✅
- `reference_detail_body.html`에 "문헌유형" 행 추가 (기존 "문헌종류" 위)

#### 1-3. reference_form2 편집 폼에 문헌유형 드롭다운 추가 ✅
- `reference_form2.html`에 `form.type` 드롭다운 추가 (문헌종류 위)

### 2단계: 데이터 정비 (추후)

#### 2-1. 기존 데이터 유형 자동 보정
- journal_title이 있으면 JN, 없으면 BK로 일괄 보정하는 management command 또는 일회성 스크립트
- 단, 보고서(RP)나 학위논문(TH)은 수동 분류 필요 → 목록 추출 후 관리자 확인

#### 2-2. import 스크립트 보정
- `import_dbpia_jkps.py`: 항상 JN (DBpia는 학술지 논문만) → 변경 불필요
- `import_jpaleodb.py`: 이미 journal_title 유무로 BK/JN 분류 → 변경 불필요
- crossref import: 메타데이터의 `type` 필드 활용 가능 (journal-article, book 등)

### 3단계: 도서 전용 필드 (선택, 추후)

현재 모델에는 저널 아티클 기준 필드만 있다 (volume, issue, pages, journal).
도서를 제대로 관리하려면 추가 필드가 필요할 수 있다:

| 필드 | 용도 | 비고 |
|------|------|------|
| publisher | 출판사 | CharField, blank=True |
| edition | 판 | CharField, blank=True |
| isbn | ISBN | CharField, blank=True |
| book_title | 단행본 제목 (book chapter인 경우) | CharField, blank=True |

→ 당장은 기존 필드만으로 운용하고, 도서 데이터가 늘어날 때 추가 검토
