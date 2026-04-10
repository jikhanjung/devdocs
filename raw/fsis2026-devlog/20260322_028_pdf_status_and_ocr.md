# 028. PDF 처리현황 페이지 및 OCR 파이프라인 실행 (2026-03-22)

## PDF 처리현황 관리자 페이지

### 기능
- `/kprdb/pdf_status/` — Professors 그룹 전용
- 시스템관리 드롭다운 메뉴에 "PDF 처리현황" 추가
- 통계 카드 4개: 텍스트 레이어, OCR, UID, 산지 추출
- 배지 클릭으로 필터링 (text_yes/no, ocr_done/failed, uid_yes/none, site_done/none 등)
- 파일 컬럼: 원본 크기 + OCR 파일 존재 시 별도 표시
- 50건 페이지네이션

### 관련 파일
- `kprdb/views.py` — `pdf_status` 뷰 추가
- `kprdb/templates/kprdb/pdf_status.html` — 템플릿
- `kprdb/urls.py` — URL 추가
- `fsis/templates/fsisnavbar.html` — 시스템관리 메뉴에 링크 추가

## OCR 파이프라인

### ocr.py 수정
- `ocr_pdf()` 기본 동작을 원본 덮어쓰기 → `_ocr.pdf` 별도 파일 생성으로 변경
- `get_ocr_path()` 함수 추가 (파일명 규칙: `원본_ocr.pdf`)
- 번체자(chi_tra) 언어 추가: eng+kor+chi_tra → eng+kor → eng → force-ocr 순서

### management commands
- `run_ocr` 커맨드 신규 생성
  - `--all`: 완료 건 재처리
  - `--limit N`: 처리 건수 제한
  - `--id N`: 특정 ID만 처리
- `run_ocr`, `check_text_layers` 양쪽에 WAL 체크포인트 자동 실행 추가

### SQLite WAL 이슈 발견 및 해결
- 호스트에서 management command로 DB 업데이트 시 WAL 파일에만 기록됨
- Docker 컨테이너가 바인드 마운트된 DB를 읽을 때 WAL 변경을 못 보는 문제
- 해결: management command 완료 시 `PRAGMA wal_checkpoint(TRUNCATE)` 자동 실행

### 호스트 환경 설정
- `ocrmypdf`, `tesseract-ocr`, `tesseract-ocr-kor`, `tesseract-ocr-chi-tra` 호스트에 설치
- venv를 requirements.txt와 동기화
- Docker 이미지에는 OCR 도구 미포함 (일괄 처리는 호스트에서 실행)

### 실행 방법
```bash
# 텍스트 레이어 확인
DATABASE_PATH=/srv/fsis2026/db.sqlite3 MEDIA_ROOT=/srv/fsis2026/uploads \
  /home/honestjung/venv/fsis2026/bin/python manage.py check_text_layers

# OCR 일괄 실행
DATABASE_PATH=/srv/fsis2026/db.sqlite3 MEDIA_ROOT=/srv/fsis2026/uploads \
  /home/honestjung/venv/fsis2026/bin/python manage.py run_ocr
```

### 처리 결과 (진행 중)
- 텍스트 레이어: 244건 있음, 82건 스캔본
- OCR: 82건 대상, 진행 중 (eng+kor+chi_tra 3언어, 건당 5~15분)

## 버전 관리

### config/version.py
- `VERSION = '0.2.17'` — 단일 소스로 버전 관리
- `config/context_processors.py` — `APP_VERSION` 템플릿 변수 제공
- navbar에 `KOFHIN v0.2.17` 표시

### 버전 이력
| 버전 | 내용 |
|------|------|
| 0.2.14 | PDF 처리현황 페이지 추가 |
| 0.2.15 | humanize 템플릿 태그 오류 수정 |
| 0.2.16 | UID 현황 추가, OCR 파일 표시 개선 |
| 0.2.17 | 버전 표기 시스템 (config/version.py) |
