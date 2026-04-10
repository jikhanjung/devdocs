# 038. DBpia 한국고생물학회지 PDF 일괄 다운로드

**날짜**: 2026-03-26
**버전**: 0.2.74

## 개요

DBpia API를 역분석하여 한국고생물학회지(1985~2014) 전체 논문 PDF를 자동 다운로드.

## DBpia API 구조

1. **호 목록**: `POST /voisListAjax` (currPage, pageSize, publicationId)
   - publicationId: `PLCT00001598` (한국고생물학회지)
   - 페이지네이션: currPage=1,11,21,...,51 / pageSize=10,20,...,60
2. **논문 목록**: `POST /journal/voisNodeList` (voisId, publicationId)
   - nodeId, 제목, 저자, 발행연도 등 반환
3. **PDF 다운로드**: `GET /pdf/pdfView.do?nodeId={nodeId}`
   - 로그인 없이 PDF 직접 다운로드 가능

## 실행 결과

- 전체 호: **55호** (1985년 1권 1호 ~ 2014년 29-30권)
- 전체 논문: **412건**
- PDF 다운로드 성공: **368건** (2.4GB)
- 실패: **44건** — 모두 섹션 구분자(논문/초록/단보 등), 실제 논문 아님

## 실패 항목 (섹션 구분자, PDF 없음)

| nodeId | 연도 | 제목 | 권호 |
|--------|------|------|------|
| NODE06357478 | 2014 | 1부 : 진화의 증거, 화석 | Vol.29-30 No.1-2 |
| NODE06357495 | 2014 | 2부 : 진화와 종교 | Vol.29-30 No.1-2 |
| NODE02096179 | 2012 | 논문 / Research Articles | Vol.28 No.1-2 |
| NODE02096186 | 2012 | 단보 / Short Note | Vol.28 No.1-2 |
| NODE02096188 | 2012 | 컬럼 / Columns | Vol.28 No.1-2 |
| NODE02096191 | 2012 | 학술회의 보고 / Conference Reports | Vol.28 No.1-2 |
| NODE02096194 | 2012 | 서평 / Book Review | Vol.28 No.1-2 |
| NODE02096196 | 2012 | 초록 / Abstracts | Vol.28 No.1-2 |
| NODE01843206 | 2011 | 논문 | Vol.27 No.2 |
| NODE01843209 | 2011 | 학술대회 발표논문 초록집 | Vol.27 No.2 |
| NODE01663549 | 2011 | 논문 | Vol.27 No.1 |
| NODE01663555 | 2011 | Review | Vol.27 No.1 |
| NODE01565673~01565678 | 2010 | 논평/논문/연구동향 | Vol.26 No.1 |
| 나머지 30건 | 2001~2009 | 논문/초록/단보/컬럼/뉴스 등 섹션 헤더 | 각 호 |

→ 실제 논문 PDF는 **100% 다운로드 성공**

## 출력 파일

- `dbpia_jkps/jkps_articles.json` — 전체 412건 메타데이터 (nodeId, 제목, 저자, 연도, 권호, PDF 다운로드 여부)
- `dbpia_jkps/pdfs/{nodeId}.pdf` — PDF 파일 368건 (프로덕션 DB 임포트 후 삭제, `.gitignore` 제외)

## 관련 스크립트

- `kprdb/management/commands/download_dbpia_pdfs.py` — DB 매칭 기반 다운로드 커맨드 (미사용, 향후 활용)

## 다음 작업

- DB의 PDF 없는 한국고생물학회지 논문 125건과 다운로드된 368건 제목 매칭
- 매칭된 PDF를 Reference.data 필드에 연결
- 한국지질학회지(DOI 기반), Geosciences Journal 등 추가 저널 PDF 확보
