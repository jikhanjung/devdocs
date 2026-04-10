# 045. 작업 워크스페이스 개선, PDF 추가 확보, 데이터 정리

**날짜**: 2026-03-27
**버전**: 0.2.83 → 0.2.85

## 1. 작업 워크스페이스 개선 (v0.2.84)

### PDF 드롭존
- PDF 없는 논문 → 왼쪽 패널 전체가 드롭존 (드래그앤드롭/클릭 업로드)
- 업로드 성공 시 즉시 새로고침하여 PDF 뷰어 표시
- `api_upload_pdf` API 추가

### 문헌종류(type2) 선택
- 오른쪽 패널 상단에 type2 드롭다운 (미분류/분류학TX/분류학T2/비분류학RV/해외화석OS)
- 변경 즉시 DB 저장 (`api_update_type2` API)
- TX/T2 선택 시에만 산지/taxa/표본 입력 폼 표시

### 기타
- 헤더의 논문 제목에 reference_detail 링크 추가
- 상태 드롭다운 변경 시 즉시 DB 저장 (기존과 동일)

## 2. 논문 편집 폼 PDF 삭제 (v0.2.85)

- reference_form2.html: 현재 PDF 옆에 "삭제" 체크박스 추가
- reference_edit2 뷰: `delete_pdf` POST 파라미터 처리

## 3. PDF 추가 확보

### handle.net 재시도 (7건 성공)
- Content-Type이 `application/octet-stream`인 경우 `%PDF` 시그니처로 판별
- 도쿄대 리포지토리(UTokyo Repository) 리뉴얼로 다운로드 가능해짐
- 교토대 리포지토리 2건은 수동 다운로드 (bitstream UUID 방식)

### 수동 PDF 확보 (4건)
- Golubic & Lee (1999), Kim & Kim (2008), Chun (2017), Kim & Lee (2017)

### 잘못된 PDF 정리
- Yabe (1922) ID=2998에 연결된 PDF 3개 → 실제 Yoon (1975, 1976×2) 논문
- 3건 새 엔트리 생성 + PDF 재연결

## 4. 데이터 정리

- 북한논문 source: `Oh et al. (2023)` → `YJOh`
- 논문 필터 드롭다운: `YJOh`로 수정
- Geosciences Journal 한국 무관 32건 삭제
- 3/24 이후 추가 논문 type2 비움 (556건)

## 5. DB 현황

| 구분 | 건수 |
|------|------|
| 전체 논문 | 1,428건 |
| PDF 보유 | 948건 (66%) |

## 수정 파일

| 파일 | 작업 |
|------|------|
| `kprdb/views_task.py` | api_upload_pdf, api_update_type2, type2_choices 컨텍스트 |
| `kprdb/templates/kprdb/task_workspace.html` | PDF 드롭존, type2 선택, 제목 링크 |
| `kprdb/urls.py` | upload_pdf, update_type2 URL 추가 |
| `kprdb/views.py` | reference_edit2에 delete_pdf 처리, source 필터 YJOh |
| `kprdb/templates/kprdb/reference_form2.html` | PDF 삭제 체크박스 |
| `dbpia_jkps/no_pdf_references.json` | handle.net 등 상태 업데이트 |
| `config/version.py` | 0.2.83 → 0.2.85 |
