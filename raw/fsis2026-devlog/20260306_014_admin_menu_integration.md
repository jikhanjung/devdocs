# 013: 시스템관리 메뉴 통합

## 작업 내용

### 1. KPRDB 관리자 메뉴를 FSIS navbar에 통합

`fsisnavbar.html` 시스템관리 드롭다운에 KPRDB 관리자 메뉴 항목 추가:
- **사용자 목록** (`kprdb:user_list_admin`)
- **FSIS 접속기록** / **KPRDB 접속기록** — 기존 단일 접속기록을 앱별로 구분
- **편집기록** (`kprdb:history_overview`)

### 2. Professors 그룹 생성 및 admin 사용자 추가

개발 DB에 'Professors' 그룹이 없어서 시스템관리 메뉴가 표시되지 않던 문제 해결.
`admin` 사용자를 Professors 그룹에 추가하여 관리자 메뉴 접근 가능하도록 설정.
