# Modan2 운영 인프라 (설정 / 로깅 / 버전 / 빌드 / CI)

Modan2 스프린트에서 기능 코드 이외의 **운영 토대**를 구축한 작업들을 모은 페이지. 설정 시스템 현대화, print → logging 마이그레이션, 버전 관리 중앙화, 빌드 시스템 / Anaconda 호환성, GitHub Actions 파이프라인이 대상이다.

## 1. QSettings → JSON 설정 시스템 (012, 2025-08-30)

### 동기

리팩토링 직후에도 설정 시스템이 **이중 구조**로 남아있었다:
- `CalibrationDialog` — `QSettings.setValue()` 직접 호출 (플랫폼 레지스트리/ini)
- 다른 Dialog — `SettingsWrapper.value()`로 **읽기만** 수행 (쓰기 실패)
- `ModanComponents` — `value()` 메서드로 설정 읽기

결과: CalibrationDialog의 설정이 실제로 저장되지 않음. 이중 시스템으로 인한 혼란.

### 해결 — SettingsWrapper 쓰기 지원

```python
class SettingsWrapper:
    def __init__(self, config, config_path):
        self.config      = config
        self.config_path = config_path

    def value(self, key, default_value):
        # 기존 읽기 기능 — QSettings 키 → JSON 경로 매핑
        ...

    def setValue(self, key, value):
        # 신규 쓰기 기능
        path = self._map_key(key)
        self._set_nested_value(path, value)

    def sync(self):
        # 설정 파일에 저장 (~/.modan2/config.json)
        with open(self.config_path, 'w') as f:
            json.dump(self.config, f, indent=2)
```

### 키 매핑 규칙

```
QSettings 키                      → JSON 경로
"WindowGeometry/MainWindow"       → ui.window_geometry.main_window
"IsMaximized/MainWindow"          → ui.is_maximized.main_window
"Calibration/Unit"                → calibration.unit
"DataPointColor/0"                → ui.data_point_colors[0]
"DataPointMarker/0"               → ui.data_point_markers[0]
"ToolbarIconSize"                 → ui.toolbar_icon_size
"PlotSize"                        → ui.plot_size
"BackgroundColor"                 → ui.background_color
"LandmarkSize/2D"                 → ui.landmark_size_2d
"LandmarkColor/2D"                → ui.landmark_color_2d
"WireframeThickness/2D"           → ui.wireframe_thickness_2d
"WireframeColor/2D"               → ui.wireframe_color_2d
"IndexSize/2D"                    → ui.index_size_2d
"IndexColor/2D"                   → ui.index_color_2d
(3D 버전도 동일 패턴)
"Language"                        → language
"LastCalibrationUnit"             → calibration.last_unit
```

### 설정 파일 스키마

```json
{
  "language": "en",
  "ui": {
    "toolbar_icon_size":  "Medium",
    "remember_geometry":  true,
    "window_geometry":    {
      "main_window":             [100, 100, 1400, 800],
      "object_dialog":           [...],
      "dataset_dialog":          [...],
      "data_exploration_window": [...],
      "dataset_analysis_window": [...],
      "export_dialog":           [...],
      "import_dialog":           [...]
    },
    "is_maximized": {
      "main_window":             false,
      "data_exploration_window": false
    },
    "landmark_size_2d":    5,
    "landmark_color_2d":   "#FF0000",
    "wireframe_thickness_2d": 1,
    "data_point_colors":   ["#FF0000", "#00FF00", "#0000FF", ...],
    "data_point_markers":  ["o", "s", "^", ...]
  },
  "calibration": {
    "unit":      "mm",
    "last_unit": "mm"
  },
  "theme": "default"
}
```

### 마이그레이션 범위

- **Modan2.py** — SettingsWrapper 확장
- **ModanDialogs.py** — 모든 Dialog의 read_settings / write_settings
- **ModanComponents.py** — 설정 읽기 일원화
- **MdAppSetup.py** — 설정 저장 로직

### 향후 고려

- 기존 사용자의 QSettings → JSON 마이그레이션 도구 (선택)
- 초기 버전에서는 두 시스템 병행 운영 → 이후 완전 제거

---

## 2. print → logging 마이그레이션 (014, 2025-08-30)

### 기존 로깅 인프라

014 세션 시작 시점에 이미 **두 개**의 로깅 구성이 존재:

**main.py**의 `setup_logging()` (표준):
```python
def setup_logging(debug: bool = False):
    level = logging.DEBUG if debug else logging.INFO
    format_str = '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    logging.basicConfig(
        level=level,
        format=format_str,
        handlers=[
            logging.StreamHandler(sys.stdout),
            logging.FileHandler('modan2.log', encoding='utf-8'),
        ],
    )
```

**MdLogger.py**의 커스텀 로거:
```python
def setup_logger(name, level=logging.INFO):
    logfile_path = os.path.join(
        mu.DEFAULT_LOG_DIRECTORY,
        mu.PROGRAM_NAME + '.' + date_str + '.log',
    )
    ...
```

로깅 파이프라인은 있었지만 **대부분의 코드는 `print`로 디버그 출력**을 하고 있어 실제 활용되지 않았다.

### 발견된 print문 (80+개)

| 파일 | 개수 | 주요 용도 |
|---|---|---|
| `Modan2.py` | 32 | 윈도우 geometry 디버그(🔍 접두사), 드래그&드롭, 객체 생성/선택 |
| `ModanDialogs.py` | 12 | calibration 추적, 에러, 형상 분석 |
| `ModanComponents.py` | 10 | 드래그&드롭, procrustes, 시각화 |
| 테스트 파일 | 25 | 테스트 스크립트 출력 |
| 기타 (MdUtils, migrate.py) | 나머지 | — |

### 마이그레이션 패턴

- 디버그용 `print("🔍 ...")` → `logger.debug(...)`
- 에러 메시지 → `logger.error(...)` + 필요 시 `logger.exception(...)`
- 정보 메시지 → `logger.info(...)`
- 테스트 출력 → 제거 또는 `logger.debug`
- 모든 모듈 상단에 `logger = logging.getLogger(__name__)` 추가

### 효과

- `--debug` 플래그로 상세 로그 제어
- `modan2.log` 파일에 구조화된 기록
- 에러 핸들링 세션(015)의 `logger.error(f"Cannot read ...")` 패턴의 전제

→ [안정성 페이지](modan2-stability.md)의 에러 핸들링 섹션이 이 로깅 시스템을 기반으로 한다.

---

## 3. 버전 관리 중앙화 (020 → 021 → 022, 2025-08-31)

### 동기

버전 정보가 여러 파일에 하드코딩되어 불일치 위험:
- `MdUtils.py`의 `PROGRAM_VERSION`
- `setup.py`
- `build.py`
- `InnoSetup/Modan2.iss`
- `__init__.py` 등

### 해결 — Single Source of Truth

신규 파일 3개:

#### `version.py` (프로젝트 루트)
```python
__version__      = "0.1.4"
__version_info__ = (0, 1, 4, None)
```

#### `version_utils.py`
- Semantic Versioning 2.0.0 검증
- 버전 파싱 / 비교 / 증가 함수
- 파일에서 버전 추출

#### `bump_version.py`
- `major` / `minor` / `patch` 범프
- Git 통합: `git commit -m "Bump version to X.Y.Z"`, `git tag vX.Y.Z`
- `CHANGELOG.md` 자동 업데이트
- 대화형 인터페이스

```bash
python bump_version.py patch   # 0.1.4 → 0.1.5
python bump_version.py minor   # 0.1.4 → 0.2.0
python bump_version.py major   # 0.1.4 → 1.0.0
```

### 기존 파일 수정

- **MdUtils.py**: `from version import __version__ as PROGRAM_VERSION` + fallback
- **setup.py**: `get_version()` 함수로 version.py 읽기
- **build.py**: version.py import + InnoSetup 템플릿 처리
- **InnoSetup/Modan2.iss.template**: 신규, `{{VERSION}}` 플레이스홀더

### 효과

- `version.py` 1개 파일만 수정하면 전체 프로젝트 반영
- 버전 검증 자동화 (SemVer 규칙)
- Git tag + CHANGELOG 추적성

---

## 4. 빌드 시스템 & Anaconda 호환성 (024, 2025-09-01)

### 문제 1 — GitHub Actions 빌드 파일 명명

macOS 빌드 파일이 `.exe` 확장자로 출력:
- `Modan2_v0.1.4_build19.exe` (156 MB) — macOS 바이너리인데 `.exe`

**원인**: `.github/workflows/build.yml` 216번째 줄의 glob이 너무 광범위:
```yaml
release-files/modan2-windows/dist/*_build*.exe  # 모든 .exe 매칭
```

**수정**:
```yaml
release-files/modan2-windows/dist/Modan2_v*_build[0-9]*.exe
```

### 문제 2 — Windows Installer 파일명 일관성

Installer만 날짜 기반(`Modan2_v0.1.4_20250831_Installer.exe`), 다른 빌드는 빌드 번호(`build19`) 기반으로 혼재.

**수정** (`Modan2.iss` / `Modan2.iss.template`):
```inno
#define BuildNumber GetEnv('BUILD_NUMBER')
OutputBaseFilename=Modan2_v{#AppVersion}_build{#BuildNumber}_Installer
```

최종 명명 규칙:
| 플랫폼 | 파일명 |
|---|---|
| Windows exe | `Modan2_v0.1.4_build19.exe` |
| Windows Installer | `Modan2_v0.1.4_build19_Installer.exe` |
| macOS | `Modan2_v0.1.4_build19_macos` |
| Linux | `Modan2_v0.1.4_build19_linux` |

### 문제 3 — Anaconda Python + pandas/platform 충돌

InnoSetup으로 설치한 Windows 버전 실행 시:

```
ValueError: failed to parse CPython sys.version:
'3.12.11 | packaged by Anaconda, Inc. | (main, Jun  5 2025, ...)'
```

**원인**: Anaconda Python이 `sys.version`에 추가 정보 삽입 → `pandas`와 `platform` 모듈이 파싱 실패. PyInstaller가 이를 번들할 때 문제 전파.

**해결**: PyInstaller runtime hook 또는 패치로 `sys.version` 문자열 정규화 (상세는 raw 024 참조).

### 추가 — `build_info.json` (030)

빌드 시 생성되어 런타임에 표시:
- 빌드 번호
- 빌드 일시
- Git commit SHA

스플래시 스크린에서 버전과 함께 표시 (030). → [안정성 페이지](modan2-stability.md)의 스플래시 섹션 참조.

---

## 5. GitHub Actions CI/CD (004, 2025-08-28 → 030, 2025-09-05)

### 초기 `test.yml` (004)

- Trigger: push / PR
- Ubuntu + Python 매트릭스
- `pip install -r requirements-dev.txt` → `pytest tests/`

### 경로 조정 (007)

파일 재구성 (`pytest.ini`, `requirements-dev.txt` → `config/`) 후 워크플로우의 경로 수정, 캐시 키 개선 (`requirements*.txt` 다양한 위치 인식).

### `build.yml` (별도 워크플로우, 024)

3-플랫폼 빌드:
- **Windows**: PyInstaller + InnoSetup
- **macOS**: PyInstaller
- **Linux**: PyInstaller

위 "빌드 시스템" 섹션의 파일명 패턴이 이 워크플로우에 적용됨.

### xvfb 통합 (030)

pytest-qt가 GitHub Actions headless 환경에서 `XIO fatal error` 발생. 해결:

```yaml
- name: Run tests with virtual display
  run: |
    xvfb-run -a -s "-screen 0 1024x768x24" \
      timeout 1500 pytest tests/
```

→ [테스트 페이지](modan2-testing.md) CI 섹션 동일 내용 참조.

---

## 관련 페이지

- [Modan2 개요](modan2-overview.md)
- [Modan2 아키텍처](modan2-architecture.md) — MdAppSetup이 설정 시스템 초기화
- [Modan2 테스트 인프라](modan2-testing.md) — CI/CD, xvfb, fixture
- [Modan2 안정성](modan2-stability.md) — 에러 핸들링이 logging 시스템 전제
- [Modan2 분석 시스템](modan2-analysis.md) — numpy matrix deprecation 처리

---
*Sources: 004 (CI/CD 초기), 005 (테스트 실행 가이드), 012 (QSettings → JSON), 014 (print → logging), 020 (버전 관리 계획), 021 (추가 제안), 022 (버전 관리 완료), 024 (빌드 시스템 + Anaconda), 030 (xvfb CI, build_info).*
