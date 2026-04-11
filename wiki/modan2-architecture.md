# Modan2 아키텍처 & MVC 리팩토링

Modan2 리팩토링 스프린트의 **코어 작업**. 2025-08-29 단 하루 동안(008 계획 → 010 마스터 플랜 → 011 구현 완료) `Modan2.py` 1,500줄 모놀리스를 **Model-View-Controller 패턴**으로 분해하여, UI와 비즈니스 로직을 시그널-슬롯으로 연결했다.

## 리팩토링 전 (문제)

- `Modan2.py` 1,500+ 줄 안에 UI, 이벤트 핸들러, 데이터 처리, DB 쿼리, 분석 로직이 혼재
- 테스트 불가능 — UI 없이는 비즈니스 로직 실행 경로가 없음
- 하드코딩된 아이콘 경로 딕셔너리 (`ICON = {}`)
- 상수/헬퍼 함수 분산
- 하나의 `run_analysis()` 함수가 1,500 라인 내부에 존재
- 설정은 `QSettings` (플랫폼 종속) + 일부 JSON 혼재

## 리팩토링 후 (6단계 마스터 플랜)

원래 [010 통합 리팩토링 계획](../raw/Modan2-devlog/20250829_010_통합_리팩토링_및_테스트_계획.md) (46KB)은 6 Phase로 설계되었다. **Phase 1–4가 완료**, Phase 5–6은 추후 작업으로 남았다.

| Phase | 내용 | 상태 |
|---|---|---|
| 1 | 테스트 안전망 구축 (UI 테스트 선작성) | ✅ (025, 009) |
| 2 | 핵심 워크플로우 테스트 | ✅ (일부, → modan2-testing.md) |
| 3 | MVC 분리 (Controller 추출) | ✅ (011) |
| 4 | 테스트 구조 새 아키텍처에 맞게 업데이트 | ✅ (011, 025) |
| 5 | 커스텀 위젯(DatasetTreeWidget, ObjectTableWidget) 완전 통합 | ⏸ 보류 |
| 6 | 플러그인 시스템 도입 | ⏸ 장기 |

## 핵심 설계 — 모듈 분리

### `main.py` (신규, 진입점)

CLI 파싱 + 스플래시 + `ApplicationSetup` 오케스트레이션 + `ModanMainWindow` 런칭으로 역할 한정.

```python
def main():
    args = parse_arguments()                       # --debug, --db, --config, --lang
    setup_logging(debug=args.debug)

    app = QApplication(sys.argv)
    app.setAttribute(Qt.AA_EnableHighDpiScaling, True)
    app.setAttribute(Qt.AA_UseHighDpiPixmaps, True)

    setup = ApplicationSetup(
        debug=args.debug,
        db_path=args.db,
        config_path=args.config,
        language=args.lang,
    )

    window = ModanMainWindow(setup.get_config())
    window.show()
    app.exec_()
```

### `MdAppSetup.py` (신규, 초기화 오케스트레이션)

```python
class ApplicationSetup:
    def __init__(self, debug=False, db_path=None, config_path=None, language="en"):
        self.debug = debug
        self.db_path = db_path or str(CONFIG_DIR / "modan2.db")
        self.config_path = config_path or str(CONFIG_DIR / "config.json")

        self._setup_directories()
        self._prepare_database()      # peewee-migrate 자동 실행 (→ MdModel)
        self._load_settings()         # JSON config 로드
        self._setup_translation()     # i18n
        self._apply_theme()
```

### `ModanController.py` (신규, 비즈니스 로직 + 시그널)

`QObject`를 상속하여 시그널-슬롯 패턴으로 UI와 통신. UI는 Controller 메서드를 호출하고, 결과는 시그널로 받는다.

```python
class ModanController(QObject):
    # 시그널
    dataset_created     = pyqtSignal(object)
    dataset_updated     = pyqtSignal(object)
    object_added        = pyqtSignal(object)
    analysis_completed  = pyqtSignal(object)
    error_occurred      = pyqtSignal(str)

    def __init__(self):
        super().__init__()
        self.current_dataset = None
        self.current_object  = None
        self._processing     = False

    # 데이터셋 CRUD
    def create_dataset(self, name: str, description: str = "", **kwargs) -> MdModel.MdDataset
    def update_dataset(self, dataset_id: int, **kwargs) -> bool
    def delete_dataset(self, dataset_id: int) -> bool

    # 객체 (specimen)
    def create_object(self, dataset: MdModel.MdDataset) -> MdModel.MdObject
    def import_objects(self, file_paths: List[str]) -> Tuple[int, List[str]]

    # 분석 (PCA/CVA/MANOVA — 상세는 modan2-analysis.md)
    def run_analysis(self, dataset=None, analysis_name="", **params) -> Optional[MdModel.MdAnalysis]
    def validate_dataset_for_analysis(self, dataset) -> bool
```

### `Modan2.py` → `ModanMainWindow` (슬림화)

1,500줄 → ~900줄. UI 구성 + Controller 시그널 수신 + 사용자 입력 라우팅만 담당.

```python
class ModanMainWindow(QMainWindow):
    def __init__(self, config=None):
        super().__init__()
        self.config = config

        # 컨트롤러 초기화
        self.controller = ModanController()
        self.setup_controller_connections()

        # 아이콘 상수 사용 (MdConstants에서 주입)
        self.setWindowIcon(QIcon(ICON_CONSTANTS['app_icon_alt']))
        self.actionNewDataset = QAction(QIcon(ICON_CONSTANTS['new_dataset']), ...)

    def setup_controller_connections(self):
        self.controller.dataset_created.connect(self.on_dataset_created)
        self.controller.object_added.connect(self.on_object_added)
        self.controller.analysis_completed.connect(self.on_analysis_completed)
        self.controller.error_occurred.connect(self.on_controller_error)

    def on_action_analyze_dataset_triggered(self):
        if not self.controller.validate_dataset_for_analysis(self.selected_dataset):
            return
        # 다이얼로그 → Controller 호출
        if ret == 1:
            self.controller.run_analysis(
                dataset=self.selected_dataset,
                analysis_name=analysis_name,
                superimposition_method=superimposition_method,
                cva_group_by=cva_group_by,
                manova_group_by=manova_group_by,
            )
```

### `MdConstants.py` + `MdHelpers.py` (신규, 중앙 상수/헬퍼)

하드코딩 제거:

```python
# MdConstants.py
ICONS = {
    'new_dataset': str(ICONS_DIR / 'M2NewDataset_1.png'),
    'new_object':  str(ICONS_DIR / 'M2NewObject_2.png'),
    'import':      str(ICONS_DIR / 'M2Import_1.png'),
    'export':      str(ICONS_DIR / 'M2Export_1.png'),
    'analysis':    str(ICONS_DIR / 'M2Analysis_1.png'),
    # ...
}

FILE_FILTERS = {
    'landmark': "Landmark Files (*.tps *.nts *.txt);;...",
    'image':    "Image Files (*.jpg *.jpeg *.png *.bmp *.tiff);;...",
    '3d_model': "3D Model Files (*.obj *.ply *.stl);;...",
}

# MdHelpers.py
def show_info(parent, message: str, title: str = "Information"): ...
def show_warning(parent, message: str, title: str = "Warning"): ...
def show_error(parent, message: str, title: str = "Error"): ...
def ask_yes_no(parent, message: str, title: str = "Confirm") -> bool: ...
```

### `MdUtils.py` 확장 (파일 I/O)

리팩토링 과정에서 파일 파싱 함수들이 공식 API가 되었다. → [안정성 페이지](modan2-stability.md)의 에러 핸들링 추가도 이 시점:

```python
def read_landmark_file(file_path: str) -> List[List[float]]:
    """TPS/NTS 자동 감지 후 라우팅"""

def read_tps_file(file_path: str) -> Dict[str, Any]: ...
def read_nts_file(file_path: str) -> Dict[str, Any]: ...

def export_dataset_to_csv(dataset, file_path: str) -> bool: ...
def export_dataset_to_excel(dataset, file_path: str) -> bool: ...
```

### `MdStatistics.py` 확장 (현대적 분석 API)

리팩토링 과정에서 분석 함수들도 모듈 밖으로 추출 → [분석 시스템 페이지](modan2-analysis.md):

```python
def do_pca_analysis(landmark_data)         -> Dict[str, Any]
def do_cva_analysis(landmark_data, groups) -> Dict[str, Any]
def do_manova_analysis(data_matrix, groups) -> Dict[str, Any]
def do_procrustes_analysis(landmark_data)  -> Dict[str, Any]
```

## 시그널-슬롯 패턴 (아키텍처 효과)

리팩토링 전에는 분석 실행 → 직접 DB 업데이트 → 직접 UI refresh가 한 함수 안에서 이루어졌다. 리팩토링 후에는:

```
[User Action]
     ↓
ModanMainWindow.on_action_analyze_dataset_triggered()
     ↓
ModanController.run_analysis(...)              ← 비즈니스 로직 (UI 모름)
     ↓
  (DB 작업, MdStatistics 호출, JSON 직렬화)
     ↓
controller.analysis_completed.emit(analysis)   ← 시그널 방출
     ↓
ModanMainWindow.on_analysis_completed(analysis) ← 슬롯 수신 → UI 갱신
```

**테스트 가능성 폭증**: Controller를 UI 없이 인스턴스화하여 비즈니스 로직 단위 테스트 가능. → [테스트 페이지](modan2-testing.md)에서 `controller` fixture, `controller_with_data` fixture 참조.

## Backward Compatibility — API 호환성

리팩토링 중에도 기존 호출 경로를 깨뜨리지 않기 위해 `run_analysis()`는 이중 시그니처 처리:

```python
def run_analysis(self, dataset=None, analysis_name="", **kwargs):
    if isinstance(dataset, str):
        # 기존 스타일: run_analysis("PCA", {params})
        analysis_type = dataset
        params = analysis_name if isinstance(analysis_name, dict) else {}
        analysis_name = params.get("name", f"{analysis_type}_Analysis")
    else:
        # 신규 스타일: run_analysis(dataset=dataset, analysis_name=name, ...)
        ...
```

## 리팩토링 결과 지표 (011 보고서)

| 항목 | 결과 |
|---|---|
| 애플리케이션 실행 | ✅ 정상 |
| 테스트 통과 | **131/170** (77%, 이전 대비 +26) |
| UI 기본 테스트 | 21/21 (100%) |
| 컨트롤러 테스트 | 28/38 (74%) |
| 모델 테스트 | 82/82 (100%) |
| UI 다이얼로그 | 0/16 (추후 업데이트 필요) |
| 워크플로우 | 0/10 (추후 업데이트 필요) |
| 메인 파일 라인 수 | 1,500 → 900 (40% 감소) |

## 관련 페이지

- [Modan2 개요](modan2-overview.md) — 프로젝트 전체 스냅샷
- [Modan2 테스트 인프라](modan2-testing.md) — Phase 1/2 + fixture 업데이트
- [Modan2 분석 시스템](modan2-analysis.md) — Controller의 `run_analysis` 후속 디버깅(023)
- [Modan2 운영 인프라](modan2-infrastructure.md) — SettingsWrapper, MdAppSetup
- [Modan2 안정성](modan2-stability.md) — MdUtils 에러 핸들링 추가

---
*Sources: 001 (동기), 008 (계획), 009 (UI 테스트 계획), 010 (마스터 플랜 46KB), 011 (구현 완료 보고서).*
