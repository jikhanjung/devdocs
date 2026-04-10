# P14 논문 처리 파이프라인 구현

**날짜**: 2026-03-07

## 작업 내용

### Phase 1: 모델 확장 + UID 생성
- Reference 모델에 4개 필드 추가: `uid`, `has_text_layer`, `ocr_status`, `site_extraction_status`
- 마이그레이션 `0002_historicalreference_has_text_layer_and_more.py` 생성/적용
- `kprdb/services/uid.py`: SCODA UID Schema v0.2 기반 UID 생성
  - trilobase `populate_uids_phase_c.py`와 동일한 정규화 로직 (normalize_title, extract_first_page)
  - DOI 우선, fingerprint(fp_v1) 폴백
- `kprdb/management/commands/generate_uids.py`: UID 일괄 생성 command

### Phase 2: PDF 텍스트 레이어 확인
- `kprdb/services/pdf_utils.py`: PyMuPDF로 텍스트 레이어 유무 확인
- `kprdb/management/commands/check_text_layers.py`: 일괄 확인 command

### Phase 3: OCR 처리
- `kprdb/services/ocr.py`: ocrmypdf 기반 OCR
  - 3가지 전략 순차 시도: eng+kor → eng → force-ocr
  - 성공 시 원본 PDF 교체

### Phase 4: 산지/표본 정보 추출
- `kprdb/services/text_extractor.py`: PyMuPDF 텍스트 추출
- `kprdb/services/coordinate_extractor.py`: 5가지 좌표 패턴 매칭 (DMS, DM, DD, 방향접두)
  - 한반도 범위 검증 (30~45°N, 120~135°E)
- `kprdb/services/specimen_extractor.py`: 15개 기관 표본번호 패턴
  - KIGAM, KPE, KNHM, NRICH, SNU, PKNU, JBNU, CBNU, CNU, JNU 등
  - type specimen, general specimen 패턴

### Phase 5: 관리자 UI
- **전체 현황 대시보드** (`ref_processing.html`): 논문/PDF/UID/텍스트/OCR/추출 현황 + 진행률 바
  - 시스템관리 > 논문처리 현황 메뉴 추가
- **논문별 처리 패널** (`reference_detail.html`): admin(Professors) 사용자 전용
  - 4단계 상태 표시 + 개별/전체 실행 버튼
  - AJAX POST API (`ref_process_step/<pk>/`)로 실시간 처리

## 변경 파일

### 신규
- `kprdb/services/__init__.py`, `uid.py`, `pdf_utils.py`, `ocr.py`
- `kprdb/services/text_extractor.py`, `coordinate_extractor.py`, `specimen_extractor.py`
- `kprdb/management/commands/generate_uids.py`, `check_text_layers.py`
- `kprdb/migrations/0002_historicalreference_has_text_layer_and_more.py`
- `kprdb/templates/kprdb/ref_processing.html`
- `devlog/20260306_P14_reference_processing_pipeline.md` (계획 문서)

### 수정
- `kprdb/models.py`: Reference 모델 4개 필드 추가
- `kprdb/views.py`: `ref_processing`, `ref_process_step` 뷰 + 처리 함수들
- `kprdb/urls.py`: `ref_processing/`, `ref_process_step/<pk>/` 경로
- `kprdb/templates/kprdb/reference_detail.html`: 처리 패널 추가
- `kprdb/templates/kprnavbar.html`: 논문처리 현황 메뉴
- `requirements.txt`: PyMuPDF 추가
- `deploy/DOCKER_VERSION`: 0.1.9 → 0.1.10

## Docker
- 0.1.10 빌드 완료 (PyMuPDF 포함)
