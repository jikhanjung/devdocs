# 014: 환경변수 분기 및 서버 데이터 동기화

**날짜**: 2026-03-06

## 작업 내용

### 1. DATABASE_PATH 환경변수 분기
- `config/settings.py`에서 DB 경로를 `DATABASE_PATH` 환경변수로 분기
- 기본값: `BASE_DIR / 'db.sqlite3'` (Docker 환경에서 그대로 동작)
- 개발 환경: `/srv/fsis2026/db.sqlite3` (프로덕션 DB 사용)

### 2. MEDIA_ROOT 환경변수 분기
- `config/settings.py`에서 uploads 경로를 `MEDIA_ROOT` 환경변수로 분기
- 기본값: `BASE_DIR / 'uploads'` (Docker 환경에서 그대로 동작)
- 개발 환경: `/srv/fsis2026/uploads` (프로덕션 파일 사용)

### 3. 프로덕션 서버 데이터 동기화
- dolfinid 서버 SSH 키 설정 (`id_ed25519_kopri_desktop_wsl`)
- `rsync`로 `/srv/fsis2026/db.sqlite3` 및 `/srv/fsis2026/uploads/` 동기화 완료
- uploads 약 8.4GB (표본 사진, 논문 PDF 등)

### 4. 표본지도 링크 수정
- `target="_blank"` 제거: 같은 탭에서 열리도록 변경
- 수정 파일: `fsisnavbar.html`, `base.html`, `index.html`

### 5. 시스템관리 메뉴 확장
- KPRDB 관리자 메뉴를 FSIS 통합 navbar에 추가
- 사용자 목록, FSIS/KPRDB 접속기록, 편집기록

## 프로덕션 서버 영향
- Docker 볼륨 마운트로 기본값 그대로 동작하므로 `.env` 변경 불필요
