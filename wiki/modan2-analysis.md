# Modan2 통계 분석 시스템 (PCA / CVA / MANOVA)

Modan2의 핵심 과학 기능. `MdStatistics.py`가 제공하는 4개의 다변량 분석(**PCA**, **CVA**, **MANOVA**, **Procrustes**) 중 PCA/CVA/MANOVA는 하나의 분석 실행에서 통합 수행된다. 2025-08-31의 [023 debugging & restructuring 세션](../raw/Modan2-devlog/20250831_023_analysis_debugging_and_restructuring.md)에서 8개의 실질 버그가 한꺼번에 해결되어, 이후 안정 상태로 전환되었다.

## 개요

| 분석 | 함수 | 입력 | 출력 |
|---|---|---|---|
| **PCA** (Principal Component Analysis) | `do_pca_analysis(landmark_data)` | `List[List[List[float]]]` (specimens × landmarks × dims) | scores, eigenvectors, eigenvalues, rotation_matrix |
| **CVA** (Canonical Variate Analysis) | `do_cva_analysis(landmark_data, groups)` | landmarks + 그룹 레이블 | canonical_variables, eigenvalues |
| **MANOVA** | `do_manova_analysis(data_matrix, groups)` | PCA-reduced data + 그룹 | 4개 통계량 (Wilks/Pillai/Hotelling-Lawley/Roy) |
| **Procrustes** | `do_procrustes_analysis(landmark_data)` | landmarks | superimposed shapes, consensus |

PCA/CVA/MANOVA는 scikit-learn + scipy 기반. Procrustes superimposition은 자체 구현.

## 통합 분석 실행 (023)

**리팩토링 전**: 각 분석이 개별 버튼/다이얼로그로 실행. 사용자가 PCA → CVA → MANOVA를 수동으로 순차 실행.

**리팩토링 후**: `ModanController.run_analysis()`에서 "PCA" 실행 시 CVA와 MANOVA도 함께 실행:

```python
# ModanController.py
if analysis_type.upper() == "PCA":
    pca_result = self._run_pca(landmarks_data, params)

    # CVA도 함께 실행 (그룹 지정 시)
    if cva_group_by is not None:
        cva_params = {..., "groups": cva_group_by, ...}
        cva_result = self._run_cva(landmarks_data, cva_params)

    # MANOVA도 함께 실행 (그룹 지정 시)
    if manova_group_by is not None:
        manova_params = {..., "groups": manova_group_by, ...}
        manova_result = self._run_manova(landmarks_data, manova_params)
```

헬퍼 함수 분리(`_run_pca`, `_run_cva`, `_run_manova`)로 각 분석의 전처리/호출/결과 정규화를 일관된 형식으로 유지.

## 023 디버깅 세션 — 8개 버그 해결

023 세션은 단일 디버깅 스프린트에서 아래 버그들을 한꺼번에 해결했다.

### 1. UI Freezing (splash 메서드 누락)

**증상**: 분석 실행 직후 메인 윈도우가 숨겨진 모달 다이얼로그처럼 완전히 멈춤.
**원인**: `ModanMainWindow`에 `set_splash` 메서드 없음 → `AttributeError`가 silently swallow 됨.
**수정**:

```python
# Modan2.py
def set_splash(self, splash):
    self.splash_screen = splash
```

### 2. 분석 결과 JSON 직렬화 누락

**증상**: 분석 결과가 DB에 저장되지만 UI에서 표시 안 됨.
**원인**: 결과가 Python 객체로만 저장되고 JSON 필드(`*_analysis_result_json`)가 비어있음.
**수정**: `ModanController.py`에서 결과를 `json.dumps()`로 직렬화:

```python
analysis.object_info_json            = json.dumps(object_info_list)
analysis.pca_analysis_result_json    = json.dumps(result['scores'])
analysis.cva_analysis_result_json    = json.dumps(cva_result.get('canonical_variables'))
analysis.manova_analysis_result_json = json.dumps(manova_result)
```

### 3. CVA 결과 키 불일치

**증상**: CVA는 성공하지만 UI에 비어 있음.
**원인**: Controller가 `cva_result['scores']`를 찾았지만 `MdStatistics`는 `'canonical_variables'` 키로 저장.
**수정**:

```python
if 'canonical_variables' in cva_result:
    analysis.cva_analysis_result_json = json.dumps(cva_result['canonical_variables'])
```

### 4. MANOVA 4 통계량 모두 계산

**증상**: MANOVA 테이블에 단일 통계량만 표시.
**수정**: `MdStatistics.do_manova_analysis()`에서 4개 표준 다변량 검정 통계량을 전부 계산하여 반환:

```python
return {
    'test_statistics': [
        {'name': "Wilks' Lambda",          'value': wilks_lambda,    'f_statistic': wilks_f,    'p_value': wilks_p,    'df_num': df_num, 'df_den': df_den},
        {'name': "Pillai's Trace",         'value': pillai_trace,    'f_statistic': pillai_f,   'p_value': pillai_p,   ...},
        {'name': "Hotelling-Lawley Trace", 'value': hl_trace,        'f_statistic': hl_f,       'p_value': hl_p,       ...},
        {'name': "Roy's Greatest Root",    'value': roy_root,        'f_statistic': roy_f,      'p_value': roy_p,      ...},
    ]
}
```

ModanComponents.py의 MANOVA 테이블 렌더러도 `stat_dict` 형식 처리를 추가하도록 수정.

### 5. 3D 데이터 차원 처리 오류 ⚠️ (가장 크리티컬)

**증상**: 72 landmarks × 3 dims = 216차원이어야 할 데이터가 144차원(X/Y만)으로 축소됨. Data Exploration에서 shape dimension mismatch.

**원인**: `do_pca_analysis()`가 landmark를 flatten할 때 `landmark[:2]`로 X, Y만 사용:

```python
# 수정 전
for landmark in landmarks:
    flat_coords.extend(landmark[:2])   # Z 버림

# 수정 후
for landmark in landmarks:
    flat_coords.extend(landmark)        # X, Y, Z 전부 사용
```

### 6. Rotation Matrix — 정방행렬 확장

**증상**: PCA eigenvectors로 rotation을 적용했지만 차원 불일치.

**원인**: `pca.components_`는 `(n_components × n_features)` 직사각 행렬. 전체 공간 회전에는 정방행렬 필요.

**수정**: 항등행렬 위에 eigenvectors를 덮어써서 `(n_features × n_features)` 정방행렬 생성:

```python
# MdStatistics.py
rotation_matrix = pca.components_.T                    # (n_features × n_components)
full_rotation_matrix = np.eye(n_features)
full_rotation_matrix[:, :rotation_matrix.shape[1]] = rotation_matrix
# 결과: (216 × 216) 정방행렬
```

### 7. 분석 객체 속성 누락

**증상**: 새 분석의 plot이 "변수 이름 없음", "dimension 불명" 상태.

**원인**: `MdAnalysis.create()` 시 `dataset`의 메타데이터 속성을 복사 안 함.

**수정**:

```python
analysis = MdModel.MdAnalysis.create(
    dataset          = self.current_dataset,
    analysis_name    = analysis_name,
    propertyname_str = self.current_dataset.propertyname_str,
    dimension        = self.current_dataset.dimension,
    wireframe        = self.current_dataset.wireframe,
    baseline         = self.current_dataset.baseline,
    polygons         = self.current_dataset.polygons,
)
```

### 8. Data Exploration shape dimension 안전장치

- `propertyname_str`이 `None`일 때 fallback
- shape dimension mismatch 시 early-return + warning log

## NumPy matrix → array 마이그레이션 (028)

MANOVA 계산에서 사용하던 deprecated `numpy.matrix`를 `numpy.array`로 교체:

```python
# 수정 전
np_data = numpy.matrix(self.data)
w = numpy.matrix(within_cov)
b = numpy.matrix(between_cov)
wi = w.getI()

# 수정 후
np_data = numpy.array(self.data)
w  = numpy.array(within_cov)
b  = numpy.array(between_cov)
wi = numpy.linalg.inv(w)
```

→ [테스트 페이지](modan2-testing.md)의 Warning 해결 섹션에서도 참조.

## 분석 사전 검증 (028)

`ModanController.validate_dataset_for_analysis()`는 dataset에 grouping variable이 없을 때 CVA/MANOVA를 차단한다 (PCA는 항상 허용):

```python
grouping_vars = dataset.get_grouping_variable_index_list()
has_grouping  = len(grouping_vars) > 0 and dataset.propertyname_str

if not has_grouping:
    show_warning(None,
        f"Dataset '{dataset.dataset_name}' has no grouping variables.\n\n"
        "CVA and MANOVA analyses require grouping variables.\n"
        "Only PCA analysis will be available.")
    return False
```

이 검증이 없던 시절에는 분석 다이얼로그가 열린 후 실행 시점에 에러로 실패 → 테스트 자동화 방해 요인.

## 동적 landmark_count 계산 (030)

DB 스키마에 `landmark_count` 컬럼이 있었지만 dataset에 할당되지 않은 경우 `AttributeError`. 030에서 동적 계산으로 전환:

```python
# ModanController.py / MdUtils.py
first_obj = dataset.object_list.first() if dataset.object_list.count() > 0 else None
landmark_count = first_obj.count_landmarks() if first_obj else 0
```

스키마 변경 없이 문제 회피 + 더 유연한 처리.

## 검증 결과

| 항목 | 상태 |
|---|---|
| PCA: 2D 데이터 | ✅ |
| PCA: 3D 데이터 216차원 | ✅ (023 수정 후) |
| CVA: 결과 테이블 | ✅ (023 수정 후) |
| MANOVA: 4 통계량 | ✅ (023 수정 후) |
| Data Exploration: 3D viewer | ✅ |
| 그룹별 색상 구분 | ✅ |

## 남은 과제

1. **`unrotate_shape`** — Data Exploration에서 PCA/CVA 공간의 점을 원본 3D 공간으로 역변환. 현재는 average shape만 표시.
2. **PCA components 영속화** — 차원 축소 공간 ↔ 원본 공간 역변환을 위해 DB에 rotation matrix 저장.

## 관련 페이지

- [Modan2 개요](modan2-overview.md)
- [Modan2 아키텍처](modan2-architecture.md) — Controller의 분석 진입점
- [Modan2 테스트 인프라](modan2-testing.md) — analysis workflow 테스트, grouping 검증
- [Modan2 운영 인프라](modan2-infrastructure.md) — JSON config, 로깅
- [Modan2 안정성](modan2-stability.md) — numpy matrix deprecation, 3D 뷰어 GLUT 호환

---
*Sources: 023 (분석 디버깅 & 재구성, 8 bugs), 011 (MVC 시그니처 변경), 028 (matrix→array, grouping 검증), 030 (landmark_count 동적 계산). MdStatistics.py 확장은 011 리팩토링 보고서에도 기록.*
