# 041. jpaleodb 데이터 수집 및 임포트 상세

**날짜**: 2026-03-25~26
**버전**: 0.2.68

## 개요

일본 고생물학 데이터베이스(jpaleodb, http://www.jpaleodb.jp)에서 한국(Korea) 및 조선(Chosen) 관련 논문 서지정보와 표본 목록을 웹 스크래핑으로 수집하여 DB에 임포트.

## 1. 데이터 수집 과정

### 1단계: 논문 검색
- jpaleodb 웹사이트에서 "Korea" 및 "Chosen"으로 논문(references) 검색
- 검색 결과를 JSON으로 저장
  - `jpaleodb/jpaleodb_korea_references.json` — "Korea" 검색 결과
  - `jpaleodb/jpaleodb_chosen_references.json` — "Chosen" 검색 결과
  - `jpaleodb/jpaleodb_korea_or_chosen_references.json` — 합본 (177건)
  - `jpaleodb/kobayashi_teiichi_references.json` — 小林貞一 논문 별도 수집

### 2단계: 표본 목록 수집
- 각 논문의 `specimen_list_url`로 표본 목록 페이지 접근
- 표본별 기재명, 분류, 산지, 지층, 시대, 소장기관, 등록번호 수집
- `jpaleodb/jpaleodb_korea_or_chosen_specimens.json` — 1,532건

### 수집 데이터 구조

**논문 (references) 필드:**
```json
{
  "authors": "YABE, H.",
  "year": 1905,
  "title": "Mesozoic plants from Korea",
  "citation_en": "YABE, H. 1905:Mesozoic plants from Korea...",
  "citation_ja": "(일본어 서지사항)",
  "refid": "1234",
  "specimen_list_url": "http://www.jpaleodb.jp/...",
  "ejournal_url": "https://www.jstage.jst.go.jp/...",
  "doi": "...",
  "uid": "jpaleodb:ref:1234"
}
```

**표본 (specimens) 필드:**
```json
{
  "data_publisher": "UMUT",
  "described_name": "Coniopteris sp.",
  "type": null,
  "taxonomy": "Plantae:",
  "locality": "Korea",
  "strata": "...",
  "geologic_time": "Mesozoic",
  "institution": "University Museum, University of Tokyo",
  "register_no": "UMUT-PP-...",
  "detail_url": "http://www.jpaleodb.jp/..."
}
```

## 2. DB 임포트

### 논문 임포트 (`import_jpaleodb` management command)

**처리 로직:**
1. JSON 로드 → UID 기반 중복 제거 (177건 → 175건)
2. 기존 DB UID와 비교 → 신규만 임포트
3. 저자 파싱: `"KOBAYASHI, T., HUKASAWA, T."` → 개별 Author 객체
   - 기존 Author의 `abbreviation_e`와 정규화 비교 → 일치하면 재사용
   - 없으면 새로 생성 (형식: `"Kobayashi T."`)
4. 저널: `title_e` 정확 매칭 → `get_or_create`
5. Reference 생성: title_e, year, volume, issue, pages, journal, doi, uid
6. `remarks`에 jpaleodb 출처 정보 기록 (refid, specimen_list_url, ejournal_url)
7. `source='jpaleodb'` 설정
8. `short_title` 자동 생성

**실행 결과:**
- Reference: 175건 생성
- Author: 신규 생성 + 기존 재사용
- Journal: 신규 생성 + 기존 재사용

### 표본 임포트 (`SpecimenCandidate` 모델)

jpaleodb 표본 1,532건을 `fsis.SpecimenCandidate` 모델에 저장:
- `reference_uid`: 논문 UID (문자열 매칭용, FK 아님)
- `described_name`: 기재 종명
- `specimen_type`: 표본 유형
- `taxonomy`: 분류 계통
- `locality`: 산지
- `strata`: 지층
- `geologic_time`: 지질시대
- `institution`: 소장 기관
- `register_no`: 등록번호
- `detail_url`: jpaleodb 상세 URL
- `source`: 'jpaleodb'

### PDF 자동 다운로드 (`download_jpaleodb_pdfs` command)

논문의 `ejournal_url`에서 PDF를 자동 다운로드:
- **J-STAGE** (30건): `/_article/` → `/_pdf/` URL 치환으로 직접 다운로드
- **handle.net** (9건): 랜딩 페이지 스크래핑으로 PDF 링크 추출 (2건 성공)
- 결과: **30건 PDF 확보** (나머지는 인증 필요)

## 3. 중복 논문

jpaleodb에서 가져온 논문 중 기존 DB와 제목 기준 5건 중복 확인:

| 기존 ID | jpaleodb ID | 저자 | 비고 |
|---------|-------------|------|------|
| 2563 | 3164 | Yi, Yun, Yoon (1998) | jpaleodb에 PDF |
| 2637 | 2897 | Choi, Huh (2016) | 기존끼리 중복 |
| 2221 | 3165 | Kim, Lee, Cheong (1999) | jpaleodb에 PDF |
| 2225 | 3167 | Yun (1999) | jpaleodb에 PDF |
| 2226 | 3166 | Yun (1999) | jpaleodb에 PDF |

→ 미병합 (추후 처리 예정)

## 관련 파일

| 파일 | 설명 |
|------|------|
| `jpaleodb/jpaleodb_korea_or_chosen_references.json` | 논문 177건 |
| `jpaleodb/jpaleodb_korea_or_chosen_specimens.json` | 표본 1,532건 |
| `jpaleodb/kobayashi_teiichi_references.json` | 小林貞一 논문 |
| `kprdb/management/commands/import_jpaleodb.py` | 논문 임포트 커맨드 |
| `kprdb/management/commands/download_jpaleodb_pdfs.py` | PDF 다운로드 커맨드 |
| `fsis/models.py` — `SpecimenCandidate` | 표본 후보 모델 |
| `fsis/templates/fsis/specimen_candidate_list.html` | 일본표본정보 목록 페이지 |
