# FSIS2026 / KOFHIN 프로젝트 개요

**FSIS2026** (내부 코드명 / Fossil Site Information System 2026) = **KOFHIN** (공식 프로덕션 이름 / **Korea Fossil Heritage Inventory**). 한반도 화석산지 정보와 표본·문헌을 통합 관리하는 Django 기반 웹 시스템. 프로덕션 URL: **https://fsis.psok.or.kr**.

기존 nkfadmin 프로젝트(Django 3.1)를 Django 5.x로 마이그레이션하면서 재구축되었다. 논문 관리, 표본 관리, 화석산지 DB, Brilha 기반 위험도 평가, PDF OCR/추출 파이프라인을 포함한다.

## 공식 사업 맥락 (PROJECT_PROPOSAL 기준)

| 항목 | 내용 |
|---|---|
| **정식 사업명** | 국내 주요 화석산지 발굴 및 활용 중장기계획 수립 |
| **발주기관** | 국가유산청 (Korea Heritage Service) |
| **수행기관** | 한국고생물학회 (Paleontological Society of Korea) |
| **제안일** | 2026-02-22 |
| **용역 기간** | 2026.03 ~ 2026.11 (**8개월**, 240일) |

### 4대 목표 (제안서)

1. **화석산지 DB 구축** — 위경도 좌표 기반, 고유 산지 ID(GeoSite-ID) 부여
2. **화석표본 DB 구축** — 모식표본 중심, 분류·층서·보존상태·소장기관 정리, 북한·해외 포함
3. **GIS 통합 플랫폼 구현** — GeoHeritage Evaluator(가칭), 위성영상 기반 Risk Score, P0~P4 우선순위
4. **중장기 로드맵 및 예산(안) 제시** — 연차별 조사·발굴·보존·연구·활용 계획

### ID 체계 — 세 DB의 연동 축

| ID | 대상 | 설명 |
|---|---|---|
| **GeoSite-ID** | 화석산지 | 고유 산지 식별자, 좌표·층서·보호현황 연동 |
| **Specimen-ID** | 화석표본 | GeoSite-ID와 연동, 소장기관·등록번호 포함 |
| **Reference-ID** | 문헌 | 표본·산지와 연결되는 근거 문헌 (DOI 포함) |

### 5단계 과업 수행계획

| 단계 | 기간 | 주요 산출물 |
|---|---|---|
| 1. 착수 + 요구사항 확정 | 착수~1개월 | 착수보고서, 세부수행계획, **데이터 표준서** |
| 2. DB 구축 데이터 수집 | 1~4개월 | 표본 DB 1차본, 화석산지 DB 1차본, **산지 분포 GIS 레이어** |
| 3. 표본 중요성 평가 기능 | 3~4개월 | 중요성 평가 운영지침, 평가결과(시범군) |
| 4. GeoHeritage Evaluator 구축 + 위험도/우선순위 도출 | 4~7개월 | Evaluator 베타버전, **발굴 우선순위 평가표**, 연차별 계획 초안 |
| 5. 관리·활용 방안 + 최종 납품 | 7~8개월 | 최종보고서, DB, Evaluator, **전국 화석지도**, 운영매뉴얼, 예산(안) |

**보고 일정**: 착수보고(계약 후 10일) → 중간보고(계약종료 4개월 전) → 완료보고(계약종료 14일 전)

## 핵심 구성 앱

| 앱 | 역할 |
|---|---|
| **fsis** | 박물관 표본 관리, 표본 사진, 화석 지도 |
| **[kprdb](kprdb.md)** | 한국 고생물학 논문 데이터베이스 (Reference-ID 소유) |
| **[ghdb](ghdb.md)** | 지질유산 데이터베이스 (별도 컨테이너, 별도 settings) |
| **[evaluator](evaluator.md)** | 화석산지 위험도 평가 (GeoHeritage Evaluator) |

## 주요 시스템

- **[PDF 처리 파이프라인](pdf-pipeline.md)** — OCR, 정규식 추출, Claude AI 보강의 3단계 파이프라인
- **[작업 관리 시스템](task-management.md)** — 교수-학생 간 논문 처리 작업 할당 및 추적
- **[지도 시스템](map-system.md)** — Kakao Maps에서 MapLibre GL JS로의 전환
- **[배포 인프라](deployment.md)** — Docker + Nginx + host-side cron, GCP 서버
- **[데이터 수집](data-import.md)** — jpaleodb, DBpia, CrossRef, Geosciences Journal, 북한 논문 등 외부 소스 임포트
- **[KOFHIN 매뉴얼](kofhin-manuals.md)** — 관리자 + 일반사용자용 운영/데이터 입력 매뉴얼
- **[북한 논문 데이터](north-korea-data.md)** — 북한 고생물학 논문 수집·저자 매칭·PDF 확보

## 기술 스택

- **Backend**: Django 5.2 LTS, SQLite (WAL 모드), Gunicorn
- **Frontend**: Bootstrap 5, Bootstrap Icons, MapLibre GL JS
- **AI/OCR**: RunPod Chandra2 OCR, Claude (CLI 또는 API)
- **Infra**: Docker, Nginx, Let's Encrypt SSL, cron 자동화 (호스트 측)
- **Libraries**: django-imagekit, django-simple-history, django-import-export, django-autocomplete-light, PyMuPDF, OpenCV

## 개발 타임라인

| 기간 | 주요 마일스톤 |
|---|---|
| 2026-02-24 | nkfadmin → fsis2026 마이그레이션, Django 5.1 업그레이드 |
| 2026-03-04~06 | 운영 배포, UI 리뉴얼, Docker화, 백업 시스템 |
| 2026-03-07~08 | [GHDB 앱](ghdb.md) 구축, HTTPS 설정 |
| 2026-03-10~14 | Django 5.2 업그레이드, 보안 설정 |
| 2026-03-22~27 | [데이터 대량 임포트](data-import.md), PDF 수집 (59%→66%) |
| 2026-03-28~30 | [RunPod OCR 배치](pdf-pipeline.md) (966건 100%), PDF 뷰어 |
| 2026-04-01~06 | [추출 파이프라인](pdf-pipeline.md) 통합, 작업 관리 개선 |
| 2026-04-07~10 | 문헌유형 확장, 고급 검색, 페이지네이션 |
| 2026-04-11 | 저널 데이터 정리, 학명 이탤릭 렌더링, 저널 목록 논문 수 표시 (068) |

## 데이터 규모 (2026-04-11 기준)

- 논문: ~1,428건
- PDF 확보: ~948건 (66%)
- OCR 처리: 966건 (100% of PDFs with text)
- AI 추출: 420건+
- 표본: 37,232건 (초기 마이그레이션 기준)
- 화석산지: 50건+ (KPRDB 임포트)
- 북한 논문: 101건 (PDF 96건)
- 스모크 테스트: 61건

## 해외 플랫폼 벤치마크 (제안서)

| 기관 | 규모 | 특징 |
|---|---|---|
| **iDigBio** (미국) | 1.5억 건 (2026.02) | 820개+ 기관 연동, 국가 차원 표본 디지털화 |
| **스미소니언 자연사박물관** (미국) | 고생물 4천만 점 / 온라인 45만 건+ | 표본 사진·분류·산지정보 웹서비스 |
| **로얄온타리오박물관** (캐나다) | 캄브리아기 화석 대형 컬렉션 | 초고해상도 이미지 + 재구성 미디어 |
| **국립자연사박물관** (영국) | 표본 기록 615만 건 / 고생물 63만 건 | 검색·다운로드 가능 오픈 데이터 포털 |

## 참고 문헌 (제안서)

- **Brilha, J. (2016)**. *Inventory and quantitative assessment of geosites and geodiversity sites*. Geoheritage 8(2): 119–134. → [위험도 평가 5개 항목 출처](evaluator.md)
- **Sevieri, G. et al. (2020)**. *A multi-hazard risk prioritisation framework for cultural heritage assets*. NHESS 20(5): 1391–1414. → [P0~P4 우선순위 매트릭스 출처](evaluator.md)
- **Oh, Y. et al. (2023)**. *Reappraisal of the Neoproterozoic to middle Paleozoic fossils of North Korea*. Geosciences Journal 27(6): 661–687. → [북한 DB 선행연구](north-korea-data.md)

---
*Sources: [raw/fsis2026-devlog/](../raw/fsis2026-devlog/) (90 files, 2026-02-24 ~ 2026-04-11) + [raw/fsis2026-docs/](../raw/fsis2026-docs/) (10 files, PROJECT_PROPOSAL, server_architecture, admin/user manuals, NK reports, import reports).*
