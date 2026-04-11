# PDF 처리 파이프라인

논문 PDF에서 좌표, 분류군, 표본번호 등을 추출하는 3단계 파이프라인.

## 파이프라인 단계

```
PDF → [1. OCR] → .pdf.json
     → [2. 정규식 추출] → .pdf.extract.json
     → [3. Claude 보강] → .pdf.extract.claude.json
```

### 1단계: OCR (RunPod Chandra2)

- **RunPod API** 기반 배치 OCR 처리
- ThreadPoolExecutor 8 workers 병렬 처리
- Cold start ~185초, 10MiB 페이로드 제한
- eng+kor+chi_tra 3개 언어 지원
- 966건 PDF, 2시간 33분에 100% 완료
- 출력: `.pdf.json` (OCR 결과 + bbox 좌표)
- **cron 자동화**: `ocr_cron.sh` 10분 주기, lockfile 안전장치

#### Chandra2 좌표계
- bbox 좌표는 0-1000 정규화 (PDF 포인트 아님)
- page_box 필드 불안정, PDF 크기는 종횡비 결정에만 사용
- 라벨 색상 코딩: Section-Header, Text, Figure, Caption, Footnote 등

### 2단계: 정규식 추출

- **40+ 정규식 패턴**으로 구조화된 데이터 추출
- 추출 대상:
  - 좌표 (DMS/DM/DD, N/S/E/W 방향 접두사) — 528건
  - 분류군 (6,090 종, 552 신종) — Systematic Paleontology 섹션 경계 감지
  - 표본번호 (40+ 기관 접두사: KIGAM, KPE, KNHM, NRICH, SNU 등) — 3,932건
- 출력: `.pdf.extract.json`

### 3단계: Claude AI 보강

- 정규식 + Claude 추출 통합, Claude가 검증/보정/보완
- **claude_augment.py**: 별도 스크립트, rate limit 핸들링 + lockfile
- 모델: Opus → **Sonnet**으로 변경 (비용 절감), cron 10분→30분
- Figure/Subfigure 분석 포함
- JSON 파싱: raw_decode fallback (extra text 처리)
- ClaudeUsageLimitError 감지, 사용량 한도 시 조기 중단 (처리 결과 보존)
- 출력: `.pdf.extract.claude.json`

#### Subfigure 분리
- OpenCV contour detection + grid fallback
- Otsu threshold + morphology 연산
- split_subfigures.py (numpy, opencv-python-headless)
- 테스트: 6/6 subfigure 감지 성공

## 파일 구조 변천

```
초기:  .pdf.json (OCR) + .claude-extract.json (Claude)
현재:  .pdf.json → .pdf.extract.json → .pdf.extract.claude.json
```
- 420건 기존 파일 마이그레이션 (migrate_claude_extract.py)
- `_best_extract_path()` 헬퍼 (`views.py`): 최선 데이터 자동 선택
  - **우선순위**: `.pdf.extract.claude.json` 있으면 사용, 없으면 `.pdf.extract.json` fallback
  - PDF 뷰어/워크스페이스/상세 페이지 모두 이 헬퍼를 거쳐 단일 진실의 원천 유지
  - 서버 아키텍처 문서(`docs/server_architecture.md`)에 공식 기록

## Figure-Caption 매칭

3단계 우선순위:
1. 번호 매칭
2. Y좌표 근접도
3. 인접 청크

966건 논문에서 78-83% 매칭률

## PDF 뷰어 (pdf_detail)

- Canvas 레이아웃/텍스트 렌더링 탭
- Figure 탭: bbox 파라미터로 크롭 표시
- OCR 텍스트 탭: 마크다운 형식 페이지별 표시
- click-drag 패닝, zoom-to-width 초기 표시
- localStorage 탭 지속성

## 처리 현황 페이지

- **pdf_status_by_year**: 연도별 OCR/정규식/AI 추출 진행률 (색상 바)
- **pdf_status**: 추출 통계 및 필터
- ocr_to_md.py: .pdf.json → 마크다운 변환 (27-40% 용량 절감)

## PDF-DB 불일치 검증

OCR 첫 페이지 메타데이터와 DB 레코드 비교:
- 14건 의심 불일치 감지 (연도 차이 >3년)
- 잘못된 파일, 중복 논문, 섹션 헤더 오인식 등 식별

## 관련 페이지

- [KPRDB](kprdb.md) — Reference 모델의 extracted_data 필드
- [작업 관리](task-management.md) — 작업 워크스페이스에서 PDF 뷰어 통합
- [데이터 수집](data-import.md) — PDF 확보 파이프라인
- [배포 인프라](deployment.md) — 호스트 측 cron + venv 구조, 파일 권한/umask
- [KOFHIN 매뉴얼](kofhin-manuals.md) — 관리자용 PDF 처리 현황 페이지

---
*Sources: 019, 028, 029, 047, 048, 049, 050, 053, 054, 055, 056, 059, 061, 063, P14, docs/server_architecture.md, docs/admin_manual.md.*
