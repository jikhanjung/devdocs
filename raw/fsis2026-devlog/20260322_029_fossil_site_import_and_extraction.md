# 029. FossilSite 확충 및 PDF 데이터 추출 파이프라인 (2026-03-22)

## PDF 데이터 추출

### extracted_data 필드 추가
- `Reference.extracted_data` JSONField 추가 (마이그레이션 0003)
- 저장 구조:
  ```json
  {
    "coordinates": [{"lat", "lng", "raw", "method", "page"}, ...],
    "specimens": [{"specimen_no", "institution", "raw", "page"}, ...],
    "total_pages": int,
    "source": "original" | "ocr"
  }
  ```

### extract_pdf_data management command
- OCR 파일 우선 사용 (`_ocr.pdf`), 없으면 원본 PDF
- 페이지별 텍스트 추출 → 좌표/표본번호 regex 매칭 → 페이지 번호 기록
- DB abstract 필드에서도 추출 (page=0, source=abstract)
- 결과: 326건 중 58건에서 데이터 추출 (좌표 41개, 표본 664개)

### 좌표 추출 패턴 개선
- `coordinate_extractor.py`에 초 기호 없는 DMS 패턴 추가 (`prefix_dms_nosec`)
- 예: `N35°57′30.5, E129°26′57.3` — 기존 패턴은 `″` 기호 필수여서 미추출
- 2건 추가 추출 (ID 2717, 2719)

### reference_detail 페이지 개선
- 추출 데이터 섹션 추가 (모든 사용자 대상)
  - UID 표시
  - 좌표 테이블: 위도, 경도, 추출 방법, 페이지 번호
  - 표본번호 테이블: 표본번호, 기관, 페이지 번호, 매칭 결과
- 표본번호 매칭: ReferenceTaxonSpecimen(KPRDB) / MuseumSpecimen(FSIS) 자동 대조
  - KPRDB 매칭: taxon 이름 표시
  - FSIS 매칭: museum_specimen_detail 링크

## UID 접두사 변경
- `scoda:bib:` → `kprdb:bib:` 로 변경
- 870건 전체 재생성, 충돌 0건

## FossilSite-Reference 관계 모델
- `FossilSiteReference` 모델 신규 (fsis 앱, 마이그레이션 0006)
- FossilSite ↔ Reference 다대다 관계 (unique_together)

## KPRDB ReferenceTaxonSpecimen → FossilSite 반영

### import_sites_from_kprdb management command
- ReferenceTaxonSpecimen의 DMS 위경도 → decimal 변환
- 한반도 범위 필터 (33°-39°N, 124°-132°E), 해외 66건 제외
- 기존 FossilSite와 근접 매칭 (0.005도 ≈ 500m 이내 → 같은 산지)
- 새 FossilSite 자동 생성 + site_code 자동 채번 (FS-2026-NNNN)
- 논문 자동 연결 (FossilSiteReference)

### 실행 결과
- 한반도 내 고유좌표: 52개
- FossilSite 신규 생성: 39건 (기존 11 + 신규 39 = 총 50건)
- 논문 연결: 59건
- 기존 11건 중 2건이 RTS 근접 매칭으로 논문 연결됨
- 나머지 9건은 RTS 매칭 없음 (P06 구현 시 다른 소스에서 생성된 것으로 추정)

### 실행 방법
```bash
# dry-run
DATABASE_PATH=/srv/fsis2026/db.sqlite3 MEDIA_ROOT=/srv/fsis2026/uploads \
  python manage.py import_sites_from_kprdb --dry-run

# 실행
DATABASE_PATH=/srv/fsis2026/db.sqlite3 MEDIA_ROOT=/srv/fsis2026/uploads \
  python manage.py import_sites_from_kprdb
```

## 중복 논문 정리
- ID 2340 삭제 (ID 2619와 동일 논문, Park et al. 2013 Sesong Formation)
- ID 2619에 상세 author_list 반영, UID 충돌 접미사 제거

## 버전 이력
| 버전 | 내용 |
|------|------|
| 0.2.18 | extracted_data 표시, 표본 매칭 |
| 0.2.19 | reference_detail 표본 매칭 UI |
| 0.2.20 | select_related 필드명 수정 (referencetaxon) |
