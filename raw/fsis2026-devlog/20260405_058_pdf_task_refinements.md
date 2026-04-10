# 058. PDF 확보 작업 상태 정리, 필터, UI 개선

**날짜**: 2026-04-05  
**버전**: 0.3.16

---

## 1. PDF 확보 상태 단순화

- `pending_pdf` 폐기 → 할당 시 바로 `finding_pdf`로 생성
- 기존 182건 `pending_pdf` → `finding_pdf` 일괄 변환
- PDF 업로드 페이지 기본 필터: `finding_pdf` (확보 중인 것만 표시)

## 2. PDF 업로드 페이지 필터

상태 필터 드롭다운 추가: 전체 / PDF확보 중 / PDF확보 완료 / PDF확보 불가

## 3. 내 작업 페이지 컬럼 변경

- "연도" 컬럼 제거 (논문 컬럼에 저자(연도) 형태로 이미 표시)
- "제목" 컬럼 추가 (50자 truncate)

## 4. OCR cron 개선

- `-newer` 타임스탬프 방식 제거, 항상 JSON 없는 PDF를 직접 탐색
- Docker `entrypoint.sh`에 `umask 002` 추가 (group 쓰기 권한)
- 기존 디렉토리 전체 `chmod g+w` 적용
- 2999.pdf (1923), 3000.pdf (1926) OCR + 정규식 추출 완료

## 5. 작업 할당 데이터 정리

- 학술대회/초록 논문 81건 → admin에 PDF 확보 할당
- jpaleodb 68건 → kchwoo9에 PDF 확보 할당
- PDF 없는 미할당 논문 30건 → 10명에게 랜덤 분배 (3건씩)
- jikhanjung에 3건 추가 할당
- 일본 문헌 3건 jikhanjung → kchwoo9 재할당
- 북한 논문(YJOh) 41건: PDF 이미 있으므로 admin 작업 삭제
- PDF 이미 있는 admin 작업 7건 추가 삭제

## 6. 추출 스크립트 출력 개선

`extract_from_ocr.py`:
- 처리 시작 시 디렉토리 경로 표시 (`=== /path/to/year ===`)
- 각 논문 처리 시 파일명 + OCR 첫 페이지 첫 줄 표시
- PDF 드래그앤드롭 시 MIME type + 확장자 이중 체크
