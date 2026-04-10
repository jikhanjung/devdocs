# 054. 통합 추출 파이프라인 및 Subfigure 분리

**날짜**: 2026-04-02
**버전**: 0.3.8 → 0.3.10

## 1. 통합 추출 파이프라인 (`extract_from_ocr.py --claude`)

기존에 분리되어 있던 regex 추출과 Claude 추출을 하나로 통합.

### 동작 흐름
1. **Regex 추출** — 좌표, taxa (systematic section), 표본번호
2. **Claude 보강** (`--claude` 옵션) — 페이지별로 OCR 텍스트 + regex 결과를 Claude에 전송
3. **Claude 역할**:
   - Regex 결과 검증 (오탐 제거, 오류 수정)
   - 누락 보강 (비표준 논문 taxa, 비표준 좌표 형식)
   - **Figure/Caption 분석** — subfigure 파싱, 각 subfigure별 완전한 description
4. **결과 통합** — 하나의 `.extract.json`에 저장
   - coordinates: 중복 제거 (lat/lng 기준)
   - taxa: Claude가 보강/수정 (nomenclatural_act 업데이트)
   - specimens: Claude 추가분 병합
   - figures: Claude에서만 추출 (subfigure 포함)

### 프롬프트 설계
- Regex 결과를 "검증해달라"는 맥락으로 전달
- Figure caption에서 subfigure (a)-(z) 또는 (1)-(N) 파싱
- **각 subfigure description이 완전하고 독립적**이어야 함
  - "a-c. Dorsal views of X" → a, b, c 각각 "Dorsal views of X"

### 모델별 비교 (3494번 논문)
| | Haiku | Opus |
|---|---|---|
| JSON 파싱 에러 | ~8건 | 1건 (타임아웃) |
| Taxa | 60 | 64 |
| Figures | 23 | 23 |
| Subfigures | 134 | 126 |

Opus가 에러 적고 보수적/정확. 기본 모델을 opus로 설정.
Sonnet은 opus보다 빠르지만 JSON 파싱 에러가 좀 더 발생.

### 실행 방법
```bash
# regex만 (빠름, 무료)
python scripts/extract_from_ocr.py /srv/fsis2026/uploads/references

# regex + Claude 보강
python scripts/extract_from_ocr.py /srv/fsis2026/uploads/references/2023 --claude --force

# 모델 지정
python scripts/extract_from_ocr.py ... --claude --claude-model sonnet
```

### 버그 수정
- Claude가 `longitude: null` 반환 시 `TypeError` → null 체크 추가

## 2. Claude 추출 처리 현황

| 연도 | 논문 | 모델 | 상태 |
|------|------|------|------|
| 2024 | 1건 | opus | 완료 |
| 2023 | 3건 | opus | 완료 |
| 2022 | 7건 | opus | 완료 |
| 2021 | 27건 | sonnet | 완료 |
| 2020~ | ~930건 | - | 미처리 |

2021-2024 합산 결과:
- 좌표: 71건
- Taxa: 1,009건 (54 신종)
- 표본: 484건
- Figures: 273건, 879 subfigures

## 3. Subfigure 분리 스크립트 (`scripts/split_subfigures.py`)

Figure 이미지를 개별 subfigure로 분리하는 도구.

### 방법 1: Contour Detection (우선)
- OpenCV Otsu threshold + morphology close + contour 검출
- 흰/검은 배경에서 분리된 사진에 효과적
- 3438 Fig. 4 (6 subfigures): **정확히 6개 감지 성공**

### 방법 2: Grid Fallback
- Contour 실패 시 expected_count 기반 균등 분할
- Aspect ratio에 맞는 최적 grid 선택 (2x3, 3x4 등)
- 불규칙 plate에서는 부정확하지만 대략적 위치 제공

### 한계
- 빽빽한 plate (사진 간 경계 없음): contour 실패 → grid fallback
- 불규칙 grid (행마다 열 수 다름): 현재 미지원
- 향후 행 단위 독립 분할 또는 SAM 모델로 개선 가능

### 실행
```bash
# extract.json의 figures에 subfigure bbox 추가
python scripts/split_subfigures.py /srv/fsis2026/uploads/references/2024/3438.pdf.extract.json --debug
```

## 4. PDF 뷰어 Figures 탭 개선

- `.extract.json`의 figures 데이터를 우선 사용 (통합 파이프라인 결과)
- `.claude-extract.json` fallback (기존 별도 추출 결과)
- Figure 번호로 OCR chunk ↔ Claude figures 매칭
- Subfigure 목록: 보라색 테마 테이블 (label + description)

## 5. 기타

- `extract_claude.py` 프롬프트에 figure/subfigure 추출 추가 (standalone 모드)
- Claude CLI 타임아웃 120→180초
- reference_detail: PDF 보기 링크 추가
- `const extractPane` 중복 선언 JS 에러 수정 (v0.3.6)

## 의존성 추가

- numpy, opencv-python-headless, scipy (subfigure 분리용)

## 수정 파일

| 파일 | 작업 |
|------|------|
| `scripts/extract_from_ocr.py` | Claude 보강 통합, figure 추출, null 체크 |
| `scripts/extract_claude.py` | figure/subfigure 프롬프트 |
| `scripts/split_subfigures.py` | **신규** — contour + grid subfigure 분리 |
| `kprdb/views.py` | pdf_detail figures 데이터 전달, crop 파라미터 |
| `kprdb/templates/kprdb/pdf_detail.html` | Figures 탭, subfigure 표시, JS 수정 |
| `kprdb/templates/kprdb/reference_detail_body.html` | PDF 보기 링크 |
| `config/version.py` | 0.3.8 → 0.3.10 |
