# Modan2 테스트 인프라

Modan2 스프린트에서 가장 큰 단일 주제. **0개 자동화 테스트 → 192 passed / 37 skipped (100% 자동화)** 까지 9일 만에 진화했다. 4개 레벨의 의존 계층, xvfb 기반 CI/CD, QMessageBox 전역 억제, pytest-qt + peewee + PyQt5를 묶는 fixture 시스템까지 구축했다.

## 진화 타임라인

| 날짜 | 파일 | 테스트 수 | 이정표 |
|---|---|---|---|
| 08-28 | 001 | — | 개선 제안: "자동화된 테스트 스위트 부재" |
| 08-28 | 002 | — | 계획 작성 |
| 08-28 | 003 | — | 계획 검토 피드백 |
| 08-28 | 004 | — | **GitHub Actions CI/CD 파이프라인 구축** |
| 08-28 | 005 | — | 테스트 실행 방법 가이드 |
| 08-28 | 006 | — | 초기 구현 리뷰 |
| 08-28 | 007 | **73** | 초기 개선 완료 보고 (58 → 73 tests, +26%). CRUD 확장, fixture 파일, 에러 핸들링, 성능 테스트 인프라, pytest marker |
| 08-29 | 009 | — | UI 테스트 계획 (pytest-qt 도입 설계) |
| 08-29 | 010 | — | 통합 리팩토링 + 테스트 마스터 플랜 (46KB) |
| 08-29 | 011 | **131** | MVC 리팩토링 완료 후 fixture 업데이트 (+58 tests, 26% pass) |
| 09-01 | 025 | — | Object Dialog UI 테스트 |
| 09-01 | 026 | — | 테스트 스위트 분석 |
| 09-01 | 027 | — | **모놀리식 → 4-레벨 모듈형 재구조화** |
| 09-01 | 028 | **192** | Warning 해결 + 완전 자동화 (수동 개입 제거) |
| 09-05 | 030 | — | xvfb-run CI XIO 에러 해결 |

## 테스트 프레임워크 스택

- **`pytest`** — 기본 러너
- **`pytest-qt`** — Qt 이벤트 루프 통합 (`qtbot`, `waitExposed`)
- **`pytest-cov`** — 커버리지 리포트
- **`pytest-mock`** — Mock 헬퍼
- **`xvfb-run`** — Linux CI 가상 디스플레이 (`xvfb-run -a -s "-screen 0 1024x768x24" pytest`)

## 4-레벨 의존 계층 (027, 2025-09-01)

초기엔 2,000+ 줄 단일 `test_dataset_dialog_direct.py` 파일에 모든 테스트가 섞여 있었다. 재구조화 후:

```
Level 1: Dataset Core        (독립)
    ↓
Level 2: Import              (Core 의존)
    ↓
Level 3: Analysis & Workflow (Core + Import 의존)
    ↓
Level 4: Legacy Integration  (전부 의존)
```

### 파일 구조

```
tests/
├── conftest.py                  # 전역 fixture + QMessageBox 억제
├── test_dataset_core.py         # Level 1 — TestDatasetCore, TestObjectCore, TestDatasetObjectIntegration
├── test_import.py               # Level 2 — TestImportDatasetDialog, TestTpsImport, TestImportWithMessageBoxHandling, TestImportEdgeCases
├── test_analysis_workflow.py    # Level 3 — TestAnalysisDialog, TestMainWindowAnalysis, TestCompleteWorkflows, TestAnalysisValidation
├── test_legacy_integration.py   # Level 4 — TestDatasetDialogEdgeCases, TestObjectDialogEdgeCases, TestImportEdgeCasesLegacy, TestIntegrationRegressionTests, TestLegacyCompatibility
├── test_ui_basic.py
├── test_controller.py
├── test_performance.py          # @pytest.mark.slow
├── fixtures/
│   ├── sample_2d.tps            # 3 landmarks × 2 specimens
│   ├── sample_2d.nts
│   ├── sample_3d.obj            # unit cube
│   ├── sample_3d.ply
│   ├── sample_3d.stl
│   ├── sample_image.jpg         # 100×100 red
│   └── invalid_file.txt         # 에러 테스트용
└── sample_data/
    └── small_sample.tps         # 6 objects × 4 landmarks (028에서 3→6 확장)
```

## 핵심 fixture (`conftest.py`)

### `qapp` (session scope)

```python
@pytest.fixture(scope='session')
def qapp():
    app = QApplication.instance() or QApplication([])
    yield app
    app.quit()
```

### `temp_db` + `mock_database` (격리)

```python
@pytest.fixture
def temp_db(tmp_path):
    return str(tmp_path / "test_modan.db")

@pytest.fixture
def mock_database(monkeypatch, temp_db):
    import MdModel
    monkeypatch.setattr(MdModel, 'DATABASE_PATH', temp_db)
    MdModel.prepare_database()
    return temp_db
```

### `main_window` (MVC 대응, 011 업데이트)

```python
@pytest.fixture
def main_window(qtbot, mock_database):
    from Modan2 import ModanMainWindow

    test_config = {
        "language": "en",
        "ui": {
            "toolbar_icon_size": "Medium",
            "remember_geometry": False,
            "window_geometry": [100, 100, 1400, 800],
            "is_maximized": False,
        },
    }
    window = ModanMainWindow(test_config)   # config 매개변수 필수 (리팩토링 후)
    qtbot.addWidget(window)
    window.show()
    with qtbot.waitExposed(window):         # waitForWindowShown → waitExposed (028)
        pass
    yield window
    window.close()
```

### `controller` / `controller_with_data`

```python
@pytest.fixture
def controller(mock_database):
    from ModanController import ModanController
    return ModanController()

@pytest.fixture
def controller_with_data(controller, sample_dataset):
    for i in range(5):
        obj = MdModel.MdObject.create(
            dataset=sample_dataset,
            object_name=f"Object_{i+1}",
            sequence=i+1,
        )
        obj.landmark_str = "\n".join([f"{i+j}.0\t{i+j+1}.0" for j in range(5)])
        obj.save()
    controller.set_current_dataset(sample_dataset)
    return controller
```

### 전역 QMessageBox 억제 (`autouse`, 028)

리팩토링 전에는 "Finished importing TPS file" 등의 팝업이 테스트를 블록시켰다. 028에서 **모든** QMessageBox API를 no-op으로 패치:

```python
@pytest.fixture(autouse=True)
def suppress_message_boxes(monkeypatch):
    monkeypatch.setattr('PyQt5.QtWidgets.QMessageBox.exec_',     lambda self: 1)
    monkeypatch.setattr('PyQt5.QtWidgets.QMessageBox.exec',      lambda self: 1)
    monkeypatch.setattr('PyQt5.QtWidgets.QMessageBox.information', lambda *a, **kw: 1)
    monkeypatch.setattr('PyQt5.QtWidgets.QMessageBox.warning',     lambda *a, **kw: 1)
    monkeypatch.setattr('PyQt5.QtWidgets.QMessageBox.critical',    lambda *a, **kw: 1)
    monkeypatch.setattr('PyQt5.QtWidgets.QMessageBox.question',    lambda *a, **kw: 1)
```

## pytest marker (007, 028)

```ini
# pytest.ini (028에서 config/ → 프로젝트 루트로 이동 — marker 인식 문제 해결)
markers =
    unit: Unit tests (fast, isolated)
    integration: Integration tests (may use database)
    slow: Slow tests (performance and stress tests)
    performance: Performance benchmarking tests
    gui: GUI tests (require Qt)
```

선택적 실행:
```bash
pytest -m "not slow"   # 일반 테스트만 (3초 내)
pytest -m "slow"       # 성능 테스트
pytest                 # 전체
```

## QMessageBox Auto-Clicking 대안 (027, Timer 기반)

028 이전에는 `QTimer`로 1초마다 visible QMessageBox를 찾아 `accept()` 호출:

```python
def setup_auto_click_messagebox(self):
    def auto_click_messagebox():
        for widget in QApplication.topLevelWidgets():
            if isinstance(widget, QMessageBox) and widget.isVisible():
                print(f"✅ Auto-clicking QMessageBox: {widget.text()}")
                widget.accept()
                return

    timer = QTimer()
    timer.timeout.connect(auto_click_messagebox)
    timer.setSingleShot(False)
    timer.start(1000)
    return timer
```

028의 전역 monkeypatch 방식이 더 단순하여 이 패턴은 일부 특수 케이스에만 남아있다.

## Warning 해결 (028)

9일 마지막 세션에서 50+ 개 deprecation warning을 10개 미만으로 감축:

| Warning | 대상 | 수정 |
|---|---|---|
| `PytestReturnNotNoneWarning` | test 함수들 | `return` 문 제거 → `assert ... is not None` |
| NumPy `matrix` deprecation | `MdStatistics.py` | `numpy.matrix` → `numpy.array`, `.getI()` → `numpy.linalg.inv()` |
| `PytestUnknownMarkWarning` | marker 인식 실패 | `config/pytest.ini` → 프로젝트 루트 `pytest.ini` 이동 |
| Peewee `related_name` deprecation | `MdModel.py` | `related_name=` → `backref=` |
| `waitForWindowShown` deprecation | `conftest.py` | `qtbot.waitForWindowShown(w)` → `with qtbot.waitExposed(w): pass` |

## CI/CD (004 → 030)

### 초기 (004, 2025-08-28)

`.github/workflows/test.yml` — Ubuntu + Python 매트릭스, `pytest tests/` 실행.

### 경로 수정 (007)

`pytest.ini`, `requirements-dev.txt`를 `config/` 하위로 이동 → 워크플로우 경로 업데이트 + 캐시 키 개선.

### xvfb 통합 (030, 2025-09-05)

GitHub Actions에서 `XIO fatal error` (X11 서버 없음)로 pytest-qt 테스트가 전부 실패. 해결:

```yaml
- name: Run tests with virtual display
  run: |
    xvfb-run -a -s "-screen 0 1024x768x24" \
      timeout 1500 pytest tests/
```

→ [안정성 페이지](modan2-stability.md) CI 호환성 섹션에서도 상세 참조.

## Grouping Variable 검증 (028)

분석 다이얼로그가 불필요하게 열리는 문제를 Controller 레벨 검증으로 차단:

```python
def _validate_dataset_for_general_analysis(self, dataset) -> bool:
    grouping_vars = dataset.get_grouping_variable_index_list()
    has_grouping_vars = len(grouping_vars) > 0 and dataset.propertyname_str

    if not has_grouping_vars:
        show_warning(None,
            f"Dataset '{dataset.dataset_name}' has no grouping variables.\n\n"
            "CVA and MANOVA analyses require grouping variables.\n"
            "Only PCA analysis will be available.")
        return False
    return True
```

## 테스트 수 추이 집계

| 날짜 | 파일 | Tests | 비고 |
|---|---|---|---|
| 스프린트 시작 | — | 0 | 수동 스크립트만 존재 |
| 08-28 (007) | test_*.py | 73 passed | + 8 slow |
| 08-29 (011) | + conftest fixture 업데이트 | 131 passed / 170 total | MVC 대응 |
| 09-01 (028) | + 워닝 해결 + 전역 suppress | **192 passed, 37 skipped** | 100% 자동화 |
| 09-05 (030) | + xvfb CI | 192+ (CI 안정) | False positive 제거 |

## 실행 명령 치트시트

```bash
# 기본
pytest tests/

# 빠른 테스트만
pytest tests/ -m "not slow"

# 성능 테스트만
pytest tests/ -m "slow"

# 특정 레벨
pytest tests/test_dataset_core.py       # Level 1
pytest tests/test_import.py             # Level 2
pytest tests/test_analysis_workflow.py  # Level 3

# 커버리지
pytest --cov=. --cov-report=html

# Linux 헤드리스 (CI와 동일)
xvfb-run -a -s "-screen 0 1024x768x24" pytest tests/
```

## 관련 페이지

- [Modan2 개요](modan2-overview.md)
- [Modan2 아키텍처](modan2-architecture.md) — Controller fixture의 배경
- [Modan2 분석 시스템](modan2-analysis.md) — Analysis workflow 테스트의 실제 대상
- [Modan2 안정성](modan2-stability.md) — xvfb CI, OpenGL 호환성

---
*Sources: 002-007 (초기 구축), 009 (UI 계획), 011 (MVC fixture 업데이트), 025 (ObjectDialog 테스트), 026 (분석), 027 (재구조화), 028 (자동화 완성 + 워닝), 030 (xvfb CI).*
