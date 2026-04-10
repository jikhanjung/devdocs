# 037. 논문 출처 필드, jpaleodb PDF 다운로드, 중복 체크

**날짜**: 2026-03-26
**버전**: 0.2.69 → 0.2.74

## 1. Reference 출처(source) 필드 추가 (v0.2.69)

- `kprdb.Reference`에 `source` CharField 추가
- `import_jpaleodb` 커맨드에서 `source='jpaleodb'` 자동 설정
- 기존 jpaleodb 레코드 175건 소급 업데이트 (프로덕션 포함)
- `reference_detail_body.html`에 UID 아래 출처 표시 (값 있을 때만)

## 2. 논문 목록 출처 필터 (v0.2.70)

- 3가지 필터 옵션: "jpaleodb 제외" (기본) / "전체 (jpaleodb 포함)" / "jpaleodb만"
- 정렬 컬럼 링크에 `&source=` 파라미터 유지

## 3. jpaleodb PDF 자동 다운로드 (v0.2.71)

- `download_jpaleodb_pdfs` management command 신규 생성
- 지원 도메인:
  - **jstage.jst.go.jp** (30건): `/_article/` → `/_pdf/` 치환으로 직접 다운로드
  - **hdl.handle.net** (9건): 랜딩 페이지 스크래핑으로 PDF 링크 추출
- dry-run 모드 (기본) / `--execute` 옵션으로 실제 다운로드
- 실행 결과: **30건 다운로드 성공**, 7건 실패 (handle.net 인증 필요)
- 프로덕션에서도 실행 완료

## 4. Paginator source 파라미터 유지 (v0.2.72~v0.2.74)

- `kprdb/templates/paginator.html`에 `&source={{source_filter}}` 추가
- `fsis/templates/paginator.html`에도 동일 추가 (실제 로드되는 템플릿)
- 페이지 이동 시 출처 필터 유지

## 5. 중복 논문 체크 결과

제목+연도+저널+페이지 기준으로 **5개 중복 그룹** 확인:

| # | 연도 | 저자 | 기존 ID | 중복 ID | 상태 |
|---|------|------|---------|---------|------|
| 1 | 1998 | Yi S., Yun H., Yoon S. | 2563 | 3164 [jpaleodb] (PDF) | 미병합 |
| 2 | 2016 | Choi B.D., Huh M. | 2637 (PDF) | 2897 (PDF) | 기존끼리 중복 |
| 3 | 1999 | Kim J.Y., Lee H.N., Cheong C.H. | 2221 | 3165 [jpaleodb] (PDF) | 미병합 |
| 4 | 1999 | Yun C.S. | 2225 | 3167 [jpaleodb] (PDF) | 미병합 |
| 5 | 1999 | Yun C.S. | 2226 | 3166 [jpaleodb] (PDF) | 미병합 |

→ 추후 병합 필요 (jpaleodb PDF를 기존 레코드로 이동 후 jpaleodb 레코드 삭제)

## 수정 파일

| 파일 | 작업 |
|------|------|
| `kprdb/models.py` | Reference.source 필드 추가 |
| `kprdb/views.py` | reference_list에 source 필터 추가 |
| `kprdb/templates/kprdb/reference_list.html` | 출처 필터 드롭다운, 정렬 링크 source 유지 |
| `kprdb/templates/kprdb/reference_detail_body.html` | 출처 표시 |
| `kprdb/management/commands/import_jpaleodb.py` | source='jpaleodb' 설정 |
| `kprdb/management/commands/download_jpaleodb_pdfs.py` | **신규** — PDF 자동 다운로드 |
| `kprdb/migrations/0009_*` | source 필드 마이그레이션 |
| `fsis/templates/paginator.html` | source 파라미터 추가 |
| `kprdb/templates/paginator.html` | source 파라미터 추가 |
