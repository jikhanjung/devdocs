# 053. PDF 뷰어 Figure 탭 및 UI 개선

**날짜**: 2026-04-02
**버전**: 0.3.2 → 0.3.8

## 1. PDF 뷰어 Figure crop 표시 (v0.3.3)

### pdf_page_image bbox crop
- `?crop=x1,y1,x2,y2` 쿼리 파라미터 추가 (0-1000 정규화 좌표)
- PIL로 해당 영역만 잘라서 PNG 반환
- 텍스트 탭에서 Figure/Image chunk → 해당 bbox 영역 crop 이미지 인라인 표시

## 2. Figures 탭 추가 (v0.3.5-0.3.8)

오른쪽 패널에 레이아웃/텍스트/**Figures**/추출 4개 탭.

### Figure-Caption 매칭 전략
966개 논문의 Figure-Caption 관계 분석 결과:
- 인접 chunk: 78%
- Y좌표 근접(<30): 83%
- 번호 매칭: 17% (Figure에 번호 있는 건 26%, Caption에 번호 있는 건 43%)

매칭 우선순위:
1. **번호 매칭**: Figure alt `Figure 3` ↔ Caption `Fig. 3` (Figure, Fig., Plate, Text-fig. 등)
2. **Y좌표 근접**: Figure 하단(bbox[3])에 가장 가까운 Caption (거리 < 50)
3. **인접 chunk**: 바로 다음 chunk가 Caption이면 사용

캡션이 없는 경우 alt text를 이탤릭으로 표시.

### 크로스 페이지 매칭
Figure와 Caption이 서로 다른 페이지에 있는 경우 자동 매칭이 어려움.
필요 시 수동 매칭으로 처리 예정.

## 3. 논문 상세 PDF 보기 링크 (v0.3.4)

- reference_detail_body.html: PDF 아이콘(다운로드) 옆에 "PDF 보기" 링크 추가
- `pdf_detail` 뷰로 연결
- SVG 인라인 → Bootstrap Icons 정리

## 4. 버그 수정

- v0.3.5: `const extractPane` 중복 선언으로 JS 전체 깨짐 → v0.3.6에서 수정
- `pass` 문 잔여 (기능 영향 없음)

## 수정 파일

| 파일 | 작업 |
|------|------|
| `kprdb/views.py` | pdf_page_image crop 파라미터, pass 정리 |
| `kprdb/templates/kprdb/pdf_detail.html` | Figures 탭, Figure-Caption 매칭, crop 이미지 |
| `kprdb/templates/kprdb/reference_detail_body.html` | PDF 보기 링크 |
| `config/version.py` | 0.3.2 → 0.3.8 |
