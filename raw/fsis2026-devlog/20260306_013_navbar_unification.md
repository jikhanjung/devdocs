# 012: FSIS/KPRDB 메뉴 통합

## 작업 내용

### 1. FSIS navbar에 KPRDB 메뉴 추가

기존 단일 링크("KPRDB") → 드롭다운 메뉴 2개로 확장:
- **논문정보**: 논문목록, 저자목록, 저널목록
- **기본정보**: 지질시대표, 지층정보, 학명정보

### 2. KPRDB에서 FSIS navbar 사용

`kprbase.html` 수정:
- `kprnavbar.html` → `fsisnavbar.html`로 변경
- `kprdb-theme` → `fsis-theme`으로 통일

KPRDB 템플릿 43개는 모두 `kprbase.html`을 상속하므로 개별 수정 불필요.

### 3. fossil_map 뷰 로그인 상태 수정

`fossil_map` 뷰에서 `user_obj`를 context에 전달하지 않아 navbar에 로그인 상태가 표시되지 않던 문제 수정.

### 4. 메뉴명 변경

- "지리정보" → "표본지도" (fsisnavbar.html, base.html, index.html)
