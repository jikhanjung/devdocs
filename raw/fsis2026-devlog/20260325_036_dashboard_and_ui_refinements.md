# 036. 대시보드 및 UI 개선

**날짜**: 2026-03-25
**버전**: 0.2.63 → 0.2.68

## 변경 내용

### 1. reference_detail UID 표시 및 카드 숨김 (v0.2.63)

- `reference_detail_body.html`: DOI 다음 줄에 UID 항목 추가
- `reference_detail.html`: "PDF 추출 데이터" 카드와 "논문 처리" 카드를 `{% comment %}` 블록으로 비활성화

### 2. 대시보드 (v0.2.64)

- `fsis/views.py` index 함수를 표본목록 리다이렉트 → 대시보드 렌더링으로 변경
- `fsis/templates/fsis/dashboard.html` 신규 생성

**통계 카드 5개**: 표본 / 논문 / 화석산지 / Taxa / 산지평가 (오늘 변경건 뱃지)
**논문정보 작업 현황**: 전체/대기/진행중/완료 숫자 + 프로그레스바
**최근 활동**: 사용자 접속 기록 최근 10건
**바로가기**: 표본목록, 화석산지, 논문목록, 산지지도, 위험도 대시보드, 내 작업

### 3. KOFHIN 로고 링크 변경 (v0.2.65)

- `fsisnavbar.html`: 로고 링크를 `museum_specimen_list` → `/fsis/` (대시보드)로 변경

### 4. 작업 현황 카드 제목 변경 (v0.2.66)

- 대시보드 "작업 현황" → "논문정보 작업 현황"으로 변경

### 5. 네비바 작업관리 메뉴 재배치 (v0.2.67)

- **Professors**: "내 작업" 단일 링크 → "작업관리" 드롭다운 (작업할당 + 내 작업)
- **일반 사용자**: 기존처럼 "내 작업" 단일 링크 유지
- 시스템관리 드롭다운에서 중복 "작업할당" 항목 제거
- fsisnavbar, kprnavbar 양쪽 적용

### 6. 네비바 구조 개편 (v0.2.68)

- 표본정보 메뉴를 드롭다운으로 변경: 표본목록, 표본지도, 변경내역, 일본표본정보
- 화석산지 메뉴를 드롭다운으로 변경: 산지목록 등

## 수정 파일

| 파일 | 작업 |
|------|------|
| `fsis/views.py` | index 함수를 대시보드로 변경 |
| `fsis/templates/fsis/dashboard.html` | **신규** — 대시보드 템플릿 |
| `fsis/templates/fsisnavbar.html` | 로고 링크, 작업관리 드롭다운, 표본/산지 메뉴 개편 |
| `kprdb/templates/kprnavbar.html` | 작업관리 드롭다운, 시스템관리 중복 제거 |
| `kprdb/templates/kprdb/reference_detail_body.html` | UID 표시 추가 |
| `kprdb/templates/kprdb/reference_detail.html` | PDF 추출/논문처리 카드 비활성화 |
| `config/version.py` | 0.2.63 → 0.2.68 |
