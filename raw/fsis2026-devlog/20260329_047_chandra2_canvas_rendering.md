# 047. Chandra2 OCR 결과의 캔버스 렌더링 주의사항

**날짜**: 2026-03-29

## 1. 배경

Chandra2 OCR 엔진은 PDF 페이지를 이미지로 변환한 후 레이아웃 분석 + OCR을 수행한다.
결과는 각 청크(chunk)마다 `label`, `content`(HTML), `bbox`(바운딩 박스)를 포함한다.
이 결과를 브라우저 캔버스에 시각적으로 렌더링할 때 여러 좌표 변환 이슈가 있었다.

## 2. bbox 좌표계

### 2.1 정규화 좌표 (0-1000)

Chandra2의 bbox는 **0-1000 범위로 정규화**된 좌표이다.

```
bbox: [x1, y1, x2, y2]  // 모두 0~1000 범위
```

- `x1, y1`: 좌상단
- `x2, y2`: 우하단
- PDF 원본의 실제 크기와 무관하게 항상 0-1000 범위

### 2.2 page_box 필드의 신뢰성 문제

JSON 결과에 `page_box` 필드가 포함되는 경우가 있으나, **신뢰할 수 없다**.

실제 발견된 문제:
- `2103.pdf`: `page_box: [0, 0, 717, 1013]` (정상 - 실제 PDF 포인트 크기)
- `2311.pdf`: `page_box: [108, 55, 527, 70]` (비정상 - 첫 번째 chunk의 bbox와 동일)

`page_box`가 잘못된 경우 페이지 높이가 `70-55=15`로 계산되어, Y 스케일이 극단적으로 커지고
모든 요소가 캔버스 상단에 뭉쳐서 표시되는 현상이 발생한다.

**결론: `page_box`에 의존하지 말고 PDF 원본에서 페이지 크기를 직접 가져와야 한다.**

## 3. 캔버스 렌더링 좌표 변환

### 3.1 필요한 정보

1. **PDF 실제 페이지 크기** (width, height in PDF points) - PyMuPDF의 `page.rect`에서 획득
2. **bbox 좌표** (0-1000 정규화)
3. **캔버스 크기** (화면 패널에 맞춤)

### 3.2 변환 순서

```
1. PDF 페이지 크기로 aspect ratio 결정
2. 패널 크기에 맞춰 캔버스 크기 계산 (aspect ratio 유지)
3. bbox(0-1000) → 캔버스 픽셀로 변환
```

### 3.3 구현 코드

```javascript
// 1. PDF 실제 크기 (서버 API에서 제공)
const pageSize = pageSizes[currentPage];  // {width, height}
const pdfW = pageSize ? pageSize.width : 1000;
const pdfH = pageSize ? pageSize.height : 1000;

// 2. 캔버스 크기 = 패널에 맞추되 PDF aspect ratio 유지
const container = canvas.parentElement;
const style = getComputedStyle(container);
const padX = parseFloat(style.paddingLeft) + parseFloat(style.paddingRight);
const padY = parseFloat(style.paddingTop) + parseFloat(style.paddingBottom);
const maxW = container.clientWidth - padX;
const maxH = container.clientHeight - padY;
const scale = Math.min(maxW / pdfW, maxH / pdfH);
canvas.width = Math.round(pdfW * scale);
canvas.height = Math.round(pdfH * scale);

// 3. bbox → 캔버스 픽셀
const sx = canvas.width / 1000;
const sy = canvas.height / 1000;
// 각 chunk에 대해:
const x = bbox[0] * sx;
const y = bbox[1] * sy;
const w = (bbox[2] - bbox[0]) * sx;
const h = (bbox[3] - bbox[1]) * sy;
```

### 3.4 주의: PDF 크기 ≠ bbox 좌표 범위

| | PDF 포인트 크기 | bbox 좌표 범위 |
|---|---|---|
| `2311.pdf` | 530 x 748 | x: 106~896, y: 55~933 |
| `2103.pdf` | 717 x 1013 | x: 147~850, y: 94~911 |

PDF 크기와 bbox 범위가 다르다. bbox는 항상 0-1000 정규화이므로, PDF 크기는 오직
**캔버스의 가로세로 비율(aspect ratio)을 결정하는 데만** 사용한다.

## 4. 서버 API에서 페이지 크기 제공

`/api/info/<pdf>/` 엔드포인트에 `page_sizes` 배열을 추가해야 한다.

```python
# views.py
doc = fitz.open(str(pdf_path))
page_sizes = []
for page in doc:
    rect = page.rect
    page_sizes.append({'width': rect.width, 'height': rect.height})
info['page_sizes'] = page_sizes
```

## 5. 작은 bbox에서의 텍스트 렌더링

Page-Header 등은 bbox 높이가 매우 작을 수 있다 (예: y1=76, y2=92 → 높이 16/1000).

### 문제점
- `h > 18` 같은 최소 높이 조건이 있으면 텍스트가 아예 안 그려짐
- `fillTextWrapped()`에 `maxH = h - 18`처럼 넘기면 음수가 되어 early return

### 해결
- 최소 높이 조건 제거
- 폰트 크기를 박스 높이에 비례하여 축소
- maxH를 최소 폰트 크기 이상으로 보장

```javascript
const fontSize = Math.min(11, h * 0.7);
ctx.font = `${fontSize}px sans-serif`;
const textY = y + Math.min(14, h * 0.8);
fillTextWrapped(ctx, text, x + 4, textY, w - 8, Math.max(h, fontSize + 2), fontSize + 3);
```

## 6. 요약 체크리스트

캔버스에 Chandra2 결과를 렌더링할 때:

- [ ] PDF 실제 페이지 크기를 API로 제공하고 있는가?
- [ ] JSON의 `page_box`에 의존하지 않고 PDF 원본 크기를 사용하는가?
- [ ] bbox를 0-1000 정규화 좌표로 취급하고 있는가?
- [ ] 캔버스 크기를 패널에 맞추되 PDF aspect ratio를 유지하는가?
- [ ] 높이가 작은 bbox에서도 텍스트가 렌더링되는가?
- [ ] 패딩을 제외한 실제 사용 가능 영역을 계산하는가?
