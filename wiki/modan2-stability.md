# Modan2 안정성 & 호환성

리팩토링 스프린트 기간 동안 축적된 **견고성 개선**을 모은 페이지. 전역 try/except 에러 핸들링, 개별 버그 수정, OpenGL/GLUT/matplotlib 플랫폼 호환성, WSL 썸네일 동기화 known issue, 스플래시 스크린 개선이 대상이다.

## 1. 전역 에러 핸들링 (015, 2025-08-30)

[20KB 세션](../raw/Modan2-devlog/20250830_015_try_catch_error_handling_implementation.md). 프로젝트 전반 스캔 후 위험도 분류와 체계적 try/except 추가.

### 문제 분석 — 발견된 위험 요소

| 유형 | 노출도 | 위험 |
|---|---|---|
| **파일 I/O** | 80% 이상에 에러 처리 누락 | 사용자 데이터 손실, silent failure |
| **DB 쿼리** | 실패 시 처리 부족 | 데이터 무결성 손상 |
| **외부 라이브러리** | `trimesh`, `PIL` 등 예외 미처리 | 예측 불가 크래시 |
| **수학 연산** | 0 나눗셈, 음수 제곱근 등 | 분석 결과 오류 |
| **타입 변환** | 문자열↔숫자 | 데이터 표시 오류 |

### 위험도 분류

**High priority** (데이터 손실 / 크래시):
1. 파일 I/O
2. DB 쿼리
3. 외부 라이브러리 호출

**Medium priority**:
1. 수학 연산
2. PyQt 작업
3. JSON 파싱

**Low priority**:
1. 타입 변환
2. 네트워크 (미사용)

### 구현 패턴 — `MdUtils.py` 랜드마크 파일 읽기

수정 전 (위험):
```python
with open(file_path, 'r') as f:
    first_line = f.readline().strip()
    if first_line.startswith('LM='):
        return read_tps_file(file_path)
```

수정 후:
```python
try:
    with open(file_path, 'r', encoding='utf-8') as f:
        first_line = f.readline().strip()
        if first_line.startswith('LM='):
            return read_tps_file(file_path)
except (FileNotFoundError, PermissionError) as e:
    logger.error(f"Cannot read landmark file {file_path}: {e}")
    raise
except UnicodeDecodeError as e:
    logger.error(f"Encoding error reading {file_path}: {e}")
    raise ValueError(
        f"Cannot decode file {file_path}. Please check file encoding."
    )
```

### TPS 파일 파싱 강화

파싱 내부에서도 `LM=`, `ID=` 라인마다 개별 try/except로 손상된 파일에 대한 명확한 에러 메시지 제공:

```python
def read_tps_file(file_path):
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            for line in f:
                line = line.strip()

                if line.startswith('LM='):
                    try:
                        landmark_count = int(line.split('=')[1])
                    except (ValueError, IndexError):
                        logger.error(f"Invalid LM line in {file_path}: {line}")
                        raise ValueError(f"Malformed TPS file: invalid LM line '{line}'")

                elif line.startswith('ID='):
                    try:
                        current_name = line.split('=')[1].strip()
                    except IndexError:
                        logger.warning(f"Invalid ID line in {file_path}: {line}")
                        current_name = "Unknown"
                # ... 좌표 파싱도 동일 패턴

    except (FileNotFoundError, PermissionError) as e:
        logger.error(f"Cannot read TPS file {file_path}: {e}")
        raise
    except UnicodeDecodeError as e:
        logger.error(f"Encoding error reading TPS file {file_path}: {e}")
        raise ValueError(f"Cannot decode TPS file {file_path}. Please check file encoding.")

    return specimens
```

NTS, Morphologika 파일도 동일 패턴 적용. → [인프라 페이지](modan2-infrastructure.md)의 logging 마이그레이션이 이 전략의 전제다.

---

## 2. 버그 수정 모음

### Window Geometry Fix (013, 2025-08-30)

리팩토링 후 윈도우 위치/크기 복원이 제대로 동작하지 않는 버그. `SettingsWrapper`가 `QRect` 대신 잘못된 타입을 반환하여 복원 실패. → [테스트 페이지](modan2-testing.md)의 `mock_settings_value`에서 `QRect(100, 100, 600, 400)`를 명시적으로 반환하는 수정이 이 버그의 후속.

### Readonly Columns Context Menu (017, 2025-08-30)

읽기 전용 컬럼에서 우클릭 컨텍스트 메뉴가 편집 옵션을 표시하던 UX 버그. 컬럼 플래그에 따라 메뉴 항목 숨김 처리.

### Fill Sequence Bug (018 → 019, 2025-08-30)

**증상**: Object sequence 번호 자동 채우기가 비연속 / 중복 생성.

**수정 사이클**:
- **018**: 버그 분석 + 수정 제안
- **019**: 구현 완료 보고, regression 테스트 추가

### Splash 메서드 누락 (023)

UI freezing → `AttributeError: set_splash`. Modan2.py에 stub 추가. → 상세는 [분석 시스템 페이지](modan2-analysis.md)의 "UI Freezing" 섹션 참조.

### Composite Detail / Timeline 관련 수정

분석 시스템 섹션에서 다룬 JSON 직렬화, CVA 키 이름, 3D dimension, rotation matrix, 분석 객체 속성 복사 등 8개는 [분석 시스템 페이지](modan2-analysis.md) 참조.

---

## 3. OpenGL / GLUT 호환성 (030, 2025-09-05)

### 문제

- **Windows / Linux**에서 `glutSolidSphere()` 호출 시 **Access Violation** 크래시
- **matplotlib** artist 제거 시 `NotImplementedError` (scatter plot collection의 경우)

### 해결 — GLUT 초기화 1회로 한정 (`ModanComponents.py`)

```python
# 모듈 레벨에서 한 번만 초기화
if GLUT_AVAILABLE and glut:
    try:
        glut.glutInit(sys.argv)
        GLUT_INITIALIZED = True
    except Exception as e:
        print(f"Warning: Failed to initialize GLUT ({e}), using fallback rendering")
        GLUT_AVAILABLE = False
        GLUT_INITIALIZED = False
```

### 해결 — matplotlib artist 안전 제거 (`ModanDialogs.py`)

```python
def safe_remove_artist(artist, ax=None):
    """Safely remove matplotlib artist from plot"""
    if artist is None:
        return
    try:
        artist.remove()
    except NotImplementedError:
        # scatter plots 등의 collection의 경우
        if ax is not None:
            if hasattr(ax, 'collections') and artist in ax.collections:
                ax.collections.remove(artist)
```

### 3D 랜드마크 인덱스 표시 복원

GLUT 크래시 문제로 비활성화되었던 3D 뷰의 랜드마크 번호 표시를 안전하게 복원:

```python
if self.show_index:
    gl.glDisable(gl.GL_LIGHTING)
    index_color = mu.as_gl_color(self.index_color)
    gl.glColor3f(*index_color)
    gl.glRasterPos3f(lm[0] + 0.05, lm[1] + 0.05, lm[2])
    font_size_list = [
        glut.GLUT_BITMAP_HELVETICA_10,
        glut.GLUT_BITMAP_HELVETICA_12,
        glut.GLUT_BITMAP_HELVETICA_18,
    ]
    if GLUT_AVAILABLE and GLUT_INITIALIZED and glut:
        try:
            for letter in list(str(i + 1)):
                glut.glutBitmapCharacter(font_size_list[int(self.index_size)], ord(letter))
        except (OSError, AttributeError):
            pass   # Fallback: 렌더링 실패 시 무시
    gl.glEnable(gl.GL_LIGHTING)
```

기본값은 `show_index = False`, ObjectDialog에서만 `True`. 향후 **QPainter overlay** 방식으로 재구현하여 GLUT 의존성 완전 제거 예정.

---

## 4. 스플래시 스크린 개선 (030)

### 즉시 표시

```python
# main.py
# Heavy import 전에 스플래시 먼저 띄우기
splash = None
if not args.no_splash:
    from MdSplashScreen import create_splash_screen
    splash = create_splash_screen(background_path)
    splash.setProgress("Starting Modan2...")
    splash.show()
    QApplication.processEvents()   # 즉시 렌더링 강제
```

### 빌드 정보 표시

- 동적 저작권 연도 (`2023-현재`)
- 빌드 번호 통합 표시
- `build_info.json` (빌드 시 생성, 런타임에 읽음) → [인프라 페이지](modan2-infrastructure.md)의 빌드 섹션 참조.

---

## 5. WSL 썸네일 동기화 (029, 2025-09-05) — Known Issue

### 증상

**WSL 환경에서만** 3D 뷰 회전 시 썸네일이 실시간 업데이트 안 됨. Native Linux에서는 정상 작동.

### 원인 분석

- **WSLg**(Wayland/X11 bridge)의 그래픽 파이프라인 특성
- X11 프로토콜 변환 과정에서 이벤트 처리 지연
- OpenGL 버퍼 → Qt pixmap 전환 경로에 race condition

### 결론

**애플리케이션 레벨에서 해결 불가.** 알려진 WSLg 이슈로 문서화, WSLg 업데이트 모니터링. Native Linux / Windows에서는 영향 없음.

---

## 6. CI/CD 안정성 (030)

### XIO Fatal Error

GitHub Actions의 pytest-qt 테스트가 headless 환경에서 실패:
```
XIO: fatal IO error 0 (Success) on X server ":0.0"
```

### 해결 — xvfb-run 가상 디스플레이

```yaml
- name: Run tests with virtual display
  run: |
    xvfb-run -a -s "-screen 0 1024x768x24" \
      timeout 1500 pytest tests/
```

- `-a` 자동 서버 번호 선택
- `-s "-screen 0 1024x768x24"` 해상도/색심도 명시
- `timeout 1500` 15분 상한 (hung test 차단)

효과: false positive 테스트 실패 완전 제거, CI 안정화. → [테스트](modan2-testing.md) / [인프라](modan2-infrastructure.md) 페이지에서도 동일 내용 참조.

---

## 7. 동적 landmark_count 계산 (030)

DB 스키마에 `landmark_count` 컬럼이 있지만 할당되지 않은 경우 `AttributeError` 발생. 스키마 변경 없이 우회:

```python
first_obj = dataset.object_list.first() if dataset.object_list.count() > 0 else None
landmark_count = first_obj.count_landmarks() if first_obj else 0
```

→ [분석 시스템 페이지](modan2-analysis.md)에도 동일 내용 기록.

---

## 커밋 히스토리 (030 세션 예시)

raw 030 파일은 커밋 단위 변경 이력을 남겼다:

1. `65f4b46` — OpenGL/GLUT + matplotlib 호환성
2. `28f06ed`, `a111788` — CI/CD XIO 에러
3. `2bb20b9`, `e2da0d1` — xvfb-run 안정화
4. `2140e60` — 스플래시 즉시 표시
5. `859f58f` — landmark_count 동적 계산
6. `5ed4c1d`, `90b5101` — WSL 썸네일 문서화
7. `edad423` — 스플래시 빌드 정보
8. `c543d9d` — 3D 랜드마크 인덱스 복원

---

## 관련 페이지

- [Modan2 개요](modan2-overview.md)
- [Modan2 아키텍처](modan2-architecture.md) — Modan2.py 슬림화 + splash 메서드
- [Modan2 테스트 인프라](modan2-testing.md) — xvfb CI, mock_settings_value
- [Modan2 분석 시스템](modan2-analysis.md) — 023 세션의 8 bugs
- [Modan2 운영 인프라](modan2-infrastructure.md) — logging 기반, build_info.json

---
*Sources: 013 (geometry), 015 (에러 핸들링), 017 (readonly ctx menu), 018/019 (fill sequence), 023 (splash 메서드), 029 (WSL 썸네일), 030 (OpenGL/GLUT, matplotlib, splash, landmark_count, xvfb).*
