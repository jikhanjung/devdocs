# Modan2 프로젝트 개요

**Modan2**는 기하학적 형태 계측학(geometric morphometrics) 연구를 위한 **PyQt5 데스크톱 애플리케이션**이다. 랜드마크 기반 2D/3D 형태 분석, 통계 분석(PCA/CVA/MANOVA/Procrustes), 다양한 파일 포맷 import/export를 제공한다. 이 위키의 Modan2 섹션은 **2025-08-28 ~ 2025-09-05 (9일)** 동안 진행된 집중 **코드 품질 현대화 스프린트**를 다룬다.

## 스냅샷

| 항목 | 값 |
|---|---|
| 언어 | Python 3.12 |
| GUI | PyQt5 (tkinter-ish 커스텀 위젯, matplotlib + OpenGL/GLUT 뷰어) |
| DB | SQLite via `peewee` ORM + `peewee-migrate` |
| 분석 | `numpy`, `scipy`, `scikit-learn` (PCA/CVA/MANOVA/Procrustes) |
| 파일 포맷 | 랜드마크: TPS, NTS, Morphologika. 3D 모델: OBJ, PLY, STL. 이미지: JPG/PNG/BMP/TIFF |
| 빌드 | PyInstaller + InnoSetup (Windows installer) |
| i18n | 한국어 + 영어 |
| 테스트 | `pytest`, `pytest-qt`, `pytest-cov`, `pytest-mock` (xvfb-run CI) |
| 버전 (sprint 시점) | 0.1.4 |

## 스프린트의 동기 (001, 2025-08-28)

Modan2는 형태 계측학 핵심 기능이 대부분 구현된 **완성도 높은 상태**였으나, 다음 리스크가 있었다:

1. **테스트 부재** — `test_script/` 폴더는 수동 검증용. 자동 회귀 테스트 스위트 없음
2. **`Modan2.py` 1,500줄 모놀리스** — UI/이벤트/데이터 로딩 혼재, 복잡도 증가
3. **플랫폼 종속 의존성** — `requirements.txt`에 Windows `.whl` 경로가 직접 박힘
4. **UI freezing** — 분석 중 `WaitCursor`로 메인 스레드 블록
5. **문서화 부족** — 개발자용 docstring/가이드 부재

이 스프린트는 기능 추가가 아니라 **유지보수성/안정성/테스트 가능성**의 토대를 다시 쌓는 데 집중했다.

## 9일 타임라인

| 날짜 | 주요 작업 | 페이지 |
|---|---|---|
| **08-28** | 개선 방향 제안(001) → pytest 도입(002-007), GitHub Actions CI/CD(004) | [테스트](modan2-testing.md), [인프라](modan2-infrastructure.md) |
| **08-29** | Modan2.py 리팩토링 계획(008) → 통합 리팩토링+테스트 마스터 플랜(010, 46KB) → **MVC 구현 완료**(011) + UI 테스트 계획(009) | [아키텍처](modan2-architecture.md) |
| **08-30** | QSettings → JSON 마이그레이션(012), window geometry fix(013), print → logging(014), **전역 try/catch 에러 핸들링**(015), readonly columns 컨텍스트 메뉴(017), fill sequence bug(018→019) | [인프라](modan2-infrastructure.md), [안정성](modan2-stability.md) |
| **08-31** | 버전 관리 중앙화(020→021→022: `version.py` + `bump_version.py`), **분석 시스템 디버깅 & 재구성**(023) | [인프라](modan2-infrastructure.md), [분석](modan2-analysis.md) |
| **09-01** | 빌드 시스템 + Anaconda 호환성(024), Object dialog UI 테스트(025), test suite 분석(026), **모놀리식 → 모듈형 테스트 재구조화**(027), 테스트 워닝 해결 & 자동화 완성(028) | [테스트](modan2-testing.md), [인프라](modan2-infrastructure.md) |
| **09-05** | WSL 썸네일 동기화 문제 문서화(029), **OpenGL/GLUT + matplotlib 호환성, xvfb CI, splash screen, 3D 랜드마크 인덱스 복원**(030) | [안정성](modan2-stability.md) |

## 스프린트 결과 (집계)

| 지표 | Before | After |
|---|---|---|
| `Modan2.py` 라인 수 | 1,500+ | ~900 (40% 감소) |
| 자동화 테스트 수 | 0 | 170+ (131 통과, 35 수정 대기) |
| 테스트 카테고리 | 없음 | unit / integration / slow / performance / gui 마커 |
| 테스트 파일 구조 | 없음 | conftest + 4 레벨 의존 계층 (core → import → workflow → legacy) |
| CI/CD | 없음 | GitHub Actions 매트릭스 + xvfb-run |
| 설정 시스템 | QSettings (플랫폼별) | JSON (`~/.modan2/config.json`) |
| 로깅 | `print` | `logging` 모듈 |
| 에러 핸들링 | 파일 I/O 80% 이상 누락 | 파일/DB/외부 라이브러리 전역 try-except |
| 버전 관리 | 하드코딩 분산 | `version.py` Single Source of Truth + `bump_version.py` |
| 아키텍처 | 모놀리스 | MVC (main + ModanController + MdAppSetup + MdConstants + MdHelpers) |

## 리팩토링 후 파일 구조

```
Modan2/
├── main.py                  # 진입점 (argparse, splash, ApplicationSetup)
├── Modan2.py                # ModanMainWindow (UI only, 900줄)
├── ModanController.py       # 비즈니스 로직 + 시그널 (dataset/object/analysis)
├── MdAppSetup.py            # ApplicationSetup (초기화 오케스트레이션)
├── MdConstants.py           # ICONS, FILE_FILTERS 상수
├── MdHelpers.py             # show_info/warning/error, ask_yes_no
├── MdModel.py               # peewee ORM (MdDataset, MdObject, MdAnalysis)
├── MdUtils.py               # 파일 I/O (TPS/NTS/Morphologika), CSV/Excel export
├── MdStatistics.py          # do_pca / do_cva / do_manova / do_procrustes
├── MdSplashScreen.py        # 즉시 표시 스플래시
├── ModanComponents.py       # 커스텀 위젯 (뷰어, 테이블 등)
├── ModanDialogs.py          # 다이얼로그 (Dataset, Object, Analysis, Import, ...)
├── version.py               # __version__ = "0.1.4" (Single Source of Truth)
├── version_utils.py         # SemVer 파싱/비교/증가
├── bump_version.py          # 버전 업데이트 스크립트 (Git tag + CHANGELOG)
├── build.py                 # PyInstaller 빌드 (version.py에서 읽음)
├── setup.py                 # (version.py에서 읽음)
├── config/
│   ├── pytest.ini
│   └── requirements-dev.txt
├── tests/
│   ├── conftest.py
│   ├── fixtures/            # sample_2d.tps, sample_3d.obj, ...
│   ├── test_dataset_core.py     # Level 1
│   ├── test_import.py           # Level 2
│   ├── test_analysis_workflow.py # Level 3
│   ├── test_legacy_integration.py # Level 4
│   ├── test_ui_basic.py
│   ├── test_controller.py
│   ├── test_performance.py      # @pytest.mark.slow
│   └── ...
├── migrations/              # peewee-migrate
└── InnoSetup/
    ├── Modan2.iss.template  # {{VERSION}} 플레이스홀더
    └── Modan2.iss           # 렌더링 결과
```

## 미완료 / 향후 과제

- **남은 35개 실패 테스트** (주로 UI 다이얼로그 + 워크플로우) 수정
- **커스텀 위젯 완전 통합** (DatasetTreeWidget, ObjectTableWidget) — Phase 5
- **플러그인 시스템** — Phase 6
- **Data Exploration의 unrotate_shape** — PCA/CVA 공간 → 원본 3D 공간 역변환
- **WSL 썸네일 동기화** — WSLg 업데이트 모니터링 (애플리케이션 측 해결 불가)
- **랜드마크 인덱스 표시** — QPainter 오버레이로 재구현하여 GLUT 의존성 제거

## 관련 페이지

- [Modan2 아키텍처 & MVC 리팩토링](modan2-architecture.md)
- [Modan2 테스트 인프라](modan2-testing.md)
- [Modan2 분석 시스템 (PCA/CVA/MANOVA)](modan2-analysis.md)
- [Modan2 운영 인프라 (설정/로깅/빌드/CI)](modan2-infrastructure.md)
- [Modan2 안정성 & 호환성](modan2-stability.md)

---
*Sources: raw/Modan2-devlog/ 전체 29 파일 (2025-08-28 ~ 2025-09-05). 001, 007, 010, 011, 012, 015, 022, 023, 027, 030 대표 샘플링.*
