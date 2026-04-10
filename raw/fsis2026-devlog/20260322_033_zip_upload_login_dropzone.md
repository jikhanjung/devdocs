# 033. ZIP 업로드, 로그인 페이지, 드래그앤드롭 사진, ghdb 사용자 관리 (2026-03-22)

## ZIP 업로드 (엑셀+사진 일괄 입력)

### fsis, ghdb 양쪽 추가
- ZIP 안에 `.xlsx` + 이미지 파일을 함께 업로드
- 이미지 파일명의 prefix가 표본번호와 매칭 (예: `GHDB-R-001_전면.jpg`)
- 미리보기: 표본정보 한 줄 + 사진 썸네일 갤러리
- 저장 시 `MuseumSpecimenPhoto` 자동 생성
- 매칭 안 된 이미지 경고 표시
- URL: fsis `/fsis/import_zip/`, ghdb `/import_zip/`

### 관련 파일
- `fsis/import_utils.py`, `ghdb/import_utils.py` — `parse_zip()`, `save_rows()` photo_map 지원
- `fsis/views_import.py`, `ghdb/views_import.py` — `import_zip`, `import_zip_preview` 뷰
- 템플릿: `import_zip.html`, `import_zip_preview.html` (각각 fsis/ghdb)

## 로그인 페이지

### fsis
- `/fsis/fsis_login/` — GET 요청 시 로그인 폼 표시 (기존: POST만 처리)
- 로그인 실패 시 에러 메시지, 성공 시 `next` URL로 리다이렉트
- `@login_required` login_url을 `/fsis/fsis_login/`으로 통일

### ghdb
- `/login/` — GET 요청 시 로그인 폼 표시
- `@login_required` login_url을 `/login/`으로 통일

## ghdb 사용자 관리

### 내 정보 / 비밀번호 변경
- `/user_info/` — 아이디, 이름, 이메일, 최근 로그인, 가입일
- `/change_password/` — Django PasswordChangeForm
- navbar 사용자 드롭다운에 링크 추가

### 관리자 사용자 관리 (staff 전용)
- `/manage/users/` — 사용자 목록 (아이디, 이름, Staff, Active, 최근 로그인)
- `/manage/users/add/` — 사용자 추가
- `/manage/users/<pk>/edit/` — 정보 수정
- `/manage/users/<pk>/password/` — 비밀번호 변경
- 시스템관리 메뉴에서 접근

### ghdb navbar 개선
- 일괄입력 메뉴: 로그인 사용자에게만 표시
- 시스템관리: Django Admin 대신 전용 사용자 목록 페이지

## 드래그앤드롭 사진 업로드

### fsis, ghdb 표본 편집 폼
- 기존 개별 파일 input → 드롭존 방식
- 이미지 드래그 또는 클릭으로 추가
- 미리보기 썸네일 + 구분/제목/설명 편집 + 제거 버튼
- 기존 등록된 사진은 썸네일 카드로 표시

### ghdb 표본유형 편집 제한
- 기존 표본 편집 시 `specimen_type` 필드 disabled

## 기타
- kprdb 페이지 타이틀: `KOFHIN — Korean Paleontologic Research Database`
- 화석산지 후보 지도 (`/kprdb/site_candidates/`)
- 추출 좌표 상태 라디오버튼 (검토/관리필요/제외/등록됨)

## 버전 이력
| 버전 | 내용 |
|------|------|
| 0.2.32 | 화석산지 후보 지도 |
| 0.2.33 | 추출 좌표 상태 라디오버튼 |
| 0.2.34 | ghdb ZIP 업로드 (미완성, 0.2.35에서 완성) |
| 0.2.35 | fsis+ghdb ZIP 업로드 |
| 0.2.36 | fsis/ghdb 로그인 페이지, ghdb 내 정보/비밀번호 변경 |
| 0.2.37 | ghdb 사용자 관리, 일괄입력 로그인 제한 |
| 0.2.38 | ghdb 사용자 관리 URL 수정 (admin/ → manage/) |
| 0.2.39 | 드래그앤드롭 사진, 표본유형 편집 제한, kprdb 타이틀 |
