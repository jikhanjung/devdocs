# P14: 논문 처리 파이프라인 — UID, OCR, 정보 추출

**작성일**: 2026-03-06
**갱신일**: 2026-03-07

## 목표

1. 논문(Reference)에 UID(Unique Identifier) 부여
2. PDF 텍스트 레이어 유무 확인 및 OCR 일괄 처리
3. 논문 내용에서 화석산지(좌표), 표본번호 등 자동 추출
4. 관리자 대시보드에서 처리 현황 한눈에 파악
5. 논문 상세 페이지에서 개별 논문의 처리 상태 표시 및 실행

---

## 처리 흐름 (논문 1건 기준)

```
[1] 텍스트 레이어 확인 (has_text_layer)
    ├── True → [3] UID 생성 → [4] 산지/표본 추출
    └── False → [2] OCR 처리 (여러 방법 시도)
                    ├── 성공 → [3] UID 생성 → [4] 산지/표본 추출
                    └── 실패 → 기록 후 수동 처리 대기
```

1. **텍스트 레이어 확인**: PyMuPDF로 PDF에 추출 가능한 텍스트가 있는지 확인
2. **OCR 처리**: 텍스트 없는 스캔본에 ocrmypdf로 OCR 적용 (eng+kor)
3. **UID 생성**: SCODA UID Schema v0.2 (DOI 우선, 없으면 fingerprint)
4. **산지/표본 추출**: 텍스트에서 좌표, 표본번호, 산지명 패턴 매칭

---

## Phase 1: Reference 모델 확장 + UID 생성 ✅ 완료

### 1-1. 모델 필드 추가 (`kprdb/models.py`) ✅

Reference 모델에 다음 필드 추가:

```python
uid = CharField(max_length=200, blank=True, null=True, unique=True, db_index=True)
has_text_layer = BooleanField(null=True)  # None=미확인, True=있음, False=스캔본
ocr_status = CharField(max_length=20, blank=True, default='',
    choices=[('', '미처리'), ('done', '완료'), ('failed', '실패'), ('skipped', '건너뜀')])
site_extraction_status = CharField(max_length=20, blank=True, default='',
    choices=[('', '미시도'), ('done', '완료'), ('no_data', '데이터없음'), ('failed', '실패')])
```

마이그레이션: `kprdb/migrations/0002_historicalreference_has_text_layer_and_more.py` ✅

### 1-2. UID 생성 로직 (`kprdb/services/uid.py`) ✅

trilobase `populate_uids_phase_c.py`와 동일한 정규화 로직 사용:

- `_normalize_title()`: NFKC, 소문자, soft hyphen 제거, `&`→and, `-/`→공백, 구두점 제거
- `_extract_first_page()`: regex `(\d+)` 매칭
- `c=` (container): 비어있으면 생략
- DOI 우선 → fingerprint 폴백

### 1-3. management command ✅

- `generate_uids.py`: `--all`, `--dry-run`, 충돌시 `-c2` 접미사
- `check_text_layers.py`: `--all`, PDF 텍스트 레이어 일괄 확인

---

## Phase 2: PDF 텍스트 레이어 확인 ✅ 완료

### 2-1. 텍스트 레이어 확인 로직 (`kprdb/services/pdf_utils.py`) ✅

```python
def check_text_layer(pdf_path):
    """PDF에 텍스트 레이어가 있는지 확인 (페이지당 평균 100자 이상)"""
```

### 2-2. management command ✅

`check_text_layers.py` — PDF가 있는 Reference만 대상, `has_text_layer` 업데이트

---

## Phase 3: OCR 처리

### 3-1. OCR 처리 로직 (`kprdb/services/ocr.py`)

텍스트 레이어가 없는 PDF에 여러 방법으로 OCR 시도:

```python
def ocr_pdf(input_path, output_path=None):
    """ocrmypdf로 OCR 처리. output_path가 None이면 원본 교체"""
    # 방법 1: ocrmypdf eng+kor
    # 방법 2: ocrmypdf eng only (한국어 인식 실패 시)
    # 방법 3: tesseract 직접 호출 (ocrmypdf 실패 시)
```

의존성: `ocrmypdf`, `tesseract-ocr`, `tesseract-ocr-kor` (이미 설치됨)

### 3-2. management command: `run_ocr.py`

- `has_text_layer=False`, `ocr_status=''`인 Reference 대상
- OCR 성공 시 원본 PDF 교체, `ocr_status='done'`, `has_text_layer=True` 갱신
- `--dry-run`, `--limit N`

---

## Phase 4: 화석산지/표본번호 추출

### 4-1. 텍스트 추출 (`kprdb/services/text_extractor.py`)

PDF에서 전문 텍스트 추출 (PyMuPDF)

### 4-2. 좌표 추출 (`kprdb/services/coordinate_extractor.py`)

DMS, decimal degree, 다양한 한국 논문 좌표 표기 패턴 매칭

### 4-3. 표본번호 추출 (`kprdb/services/specimen_extractor.py`)

기관별 패턴: KIGAM, KPE, KNHM, NRICH, SNU, PKNU 등

### 4-4. 추출 결과 저장 모델 (`ReferenceExtraction`)

```python
class ReferenceExtraction(models.Model):
    reference = ForeignKey(Reference, on_delete=CASCADE, related_name='extractions')
    extraction_type = CharField(choices=[('coordinate','좌표'),('specimen','표본번호'),('locality','산지명')])
    raw_text = TextField()
    latitude = FloatField(null=True)
    longitude = FloatField(null=True)
    specimen_no = CharField(max_length=100, blank=True)
    page_number = IntegerField(null=True)
    confidence = FloatField(default=1.0)
    verified = BooleanField(default=False)
    created_on = DateTimeField(auto_now_add=True)
```

### 4-5. management command: `extract_info.py`

- 텍스트 레이어가 있거나 OCR 완료된 PDF 대상
- `site_extraction_status` 업데이트

---

## Phase 5: 관리자 대시보드 + 논문별 처리 UI

### 5-1. 전체 현황 대시보드 (`kprdb:ref_processing`) ✅ 완료

`kprdb/templates/kprdb/ref_processing.html` — Professors 그룹 전용

표시 항목: 전체 논문/PDF/UID/텍스트레이어/OCR/산지추출 현황 + 진행률 바

### 5-2. 논문 상세 페이지 처리 패널 (신규)

`reference_detail.html`에서 admin(Professors) 사용자에게 처리 상태 및 실행 버튼 표시:

```
┌─ 논문 처리 ──────────────────────────────────────────────┐
│                                                          │
│  Step 1: 텍스트 레이어  ✅ 있음         [다시 확인]       │
│  Step 2: OCR 처리       ⏭ 건너뜀       [실행]           │
│  Step 3: UID 생성       ✅ scoda:bib:doi:10.xxx  [생성]  │
│  Step 4: 산지 추출      ⬜ 미시도       [추출]           │
│                                                          │
│  [전체 실행]  ← 1→2→3→4 순차 실행                        │
└──────────────────────────────────────────────────────────┘
```

구현 방식:
- `reference_detail` 뷰에서 admin 여부 전달
- AJAX POST `/kprdb/ref_process_step/<pk>/` 로 개별 스텝 실행
- `action` 파라미터: `check_text`, `run_ocr`, `generate_uid`, `extract_sites`, `run_all`
- 각 스텝은 동기 실행 (단일 논문이므로 빠름), JSON 응답으로 결과 반환
- `run_all`: 처리 흐름에 따라 순차 실행, 이전 스텝 실패 시 중단

### 5-3. API 엔드포인트

```python
# kprdb/urls.py
path('ref_process_step/<int:pk>/', views.ref_process_step, name='ref_process_step'),

# kprdb/views.py
@login_required
@require_POST
def ref_process_step(request, pk):
    """개별 논문 처리 스텝 실행 (AJAX)"""
    # Professors 그룹 확인
    # action에 따라 처리
    # JSON 응답: {ok: true, step: 'check_text', result: '텍스트 레이어 있음', ...}
```

---

## 구현 순서 (업데이트)

```
Phase 1 (모델+UID)        ✅ 완료
Phase 2 (텍스트레이어)     ✅ 완료 (command 구현, 미테스트)
Phase 5 (전체 대시보드)    ✅ 완료

Phase 5b (논문별 처리 UI)  ← 현재 작업
    ├── reference_detail.html 처리 패널 추가
    ├── ref_process_step API 구현
    └── 프론트엔드 AJAX 처리

Phase 3 (OCR)             ← Phase 5b에서 호출할 서비스 구현
Phase 4 (정보추출)         ← Phase 5b에서 호출할 서비스 구현
```

## 파일 구조

```
kprdb/
├── services/
│   ├── __init__.py              ✅
│   ├── uid.py                   ✅ (trilobase Phase C 호환)
│   ├── pdf_utils.py             ✅
│   ├── ocr.py                   ← Phase 3
│   ├── text_extractor.py        ← Phase 4
│   ├── coordinate_extractor.py  ← Phase 4
│   └── specimen_extractor.py    ← Phase 4
├── management/commands/
│   ├── generate_uids.py         ✅
│   ├── check_text_layers.py     ✅
│   ├── run_ocr.py               ← Phase 3
│   └── extract_info.py          ← Phase 4
└── templates/kprdb/
    ├── ref_processing.html      ✅ (전체 현황 대시보드)
    └── reference_detail.html    ← 처리 패널 추가 예정
```

## 추가 의존성

| 패키지 | 용도 | 설치 |
|--------|------|------|
| `PyMuPDF` (fitz) | PDF 텍스트 레이어 확인, 텍스트 추출 | `pip install pymupdf` |
| `ocrmypdf` | OCR 처리 | `pip install ocrmypdf` + `apt install tesseract-ocr tesseract-ocr-kor` |
