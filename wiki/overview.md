# FSIS2026 Project Overview

**FSIS2026** (Fossil Site Information System 2026)은 한국 고생물학 화석산지 정보를 관리하는 Django 기반 웹 애플리케이션이다. 기존 nkfadmin 프로젝트(Django 3.1)를 Django 5.x로 마이그레이션하면서 시작되었으며, 논문 관리, 표본 관리, 화석산지 평가, PDF 처리 파이프라인 등 다양한 기능을 포함한다.

## 핵심 구성 앱

| 앱 | 역할 |
|---|---|
| **fsis** | 박물관 표본 관리, 표본 사진, 화석 지도 |
| **[kprdb](kprdb.md)** | 한국 고생물학 논문 데이터베이스 (Korean Paleontology Reference Database) |
| **[ghdb](ghdb.md)** | 2026 지질유산 데이터베이스 (Geological Heritage Database) |
| **[evaluator](evaluator.md)** | 화석산지 위험도 평가 (Brilha 모델 기반) |

## 주요 시스템

- **[PDF 처리 파이프라인](pdf-pipeline.md)** — OCR, 정규식 추출, Claude AI 보강의 3단계 파이프라인
- **[작업 관리 시스템](task-management.md)** — 교수-학생 간 논문 처리 작업 할당 및 추적
- **[지도 시스템](map-system.md)** — Kakao Maps에서 MapLibre GL JS로의 전환
- **[배포 인프라](deployment.md)** — Docker 컨테이너화, Nginx 프록시, 자동 백업
- **[데이터 수집](data-import.md)** — jpaleodb, DBpia, CrossRef 등 외부 소스 임포트

## 기술 스택

- **Backend**: Django 5.2 LTS, SQLite (WAL 모드), Gunicorn
- **Frontend**: Bootstrap 5, Bootstrap Icons, MapLibre GL JS
- **AI/OCR**: RunPod Chandra2 OCR, Claude API (Sonnet/Opus)
- **Infra**: Docker, Nginx, Let's Encrypt SSL, cron 자동화
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

## 데이터 규모 (2026-04-10 기준)

- 논문: ~1,428건
- PDF 확보: ~948건 (66%)
- OCR 처리: 966건 (100% of PDFs with text)
- AI 추출: 420건+
- 표본: 37,232건 (초기 마이그레이션 기준)
- 화석산지: 50건+ (KPRDB 임포트)
- 스모크 테스트: 61건

---
*Sources: [raw/fsis2026-devlog/](../raw/fsis2026-devlog/) (89 devlog files, 2026-02-24 ~ 2026-04-10)*
