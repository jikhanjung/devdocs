# Treatise 소스 추출

Treatise on Invertebrate Paleontology에서 분류 체계를 추출하는 과정.

## 대상 문헌

| 문헌 | 연도 | 범위 | 상태 |
|---|---|---|---|
| **Treatise Part O (1959)** | 1959 | 삼엽충 전체 | 추출 완료 |
| **Treatise Part O Revised** | 1997 (기존 2004 오표기) | Agnostida, Redlichiida 등 | 추출 완료 |
| **기타 TSF** | 다양 | 14+ 추가 분류군 | 28/31 완료 |

## Treatise 1959 추출

### OCR → JSON 추출 (109)
- 수동 상위 분류 입력 (Order, Suborder, Superfamily, Family)
- OCR genus 추출: 정규식 기반
- fuzzy matching으로 기존 DB와 대조: 1,152 genera 매칭
- 이름 정규화

### TXT 파이프라인 (116)
- parse_treatise_txt.py: TXT → JSON 변환
- rank 키워드 파싱
- 이름 정규화
- EODISCINA_ID 하드코딩 버그 수정

### 데이터 정제 (117)
대규모 정제:
- 25 타이포 수정
- 14 저자 악센트 복원
- 22 형식 수정
- 파서 개선
- 결과: 8,331 assertions, 8,904 edges

### 연도 수정 (132)
- Treatise 발행 연도 2004 → 1997로 수정 (코드베이스 전체)
- 프로필명: treatise2004 → treatise1997
- 소스 파일, 빌드 스크립트, 문서 일괄 업데이트

## Treatise 1997 (구 2004) 추출

### 대상 (102)
- Chapter 4: Agnostida (pp. 331-403) — 2 suborders, 4 superfamilies, 17 families, 160 genera
- Chapter 5: Redlichiida (pp. 404-481) — 2 suborders, 6 superfamilies, 23 families, 170 genera

### JSON 형식
- 중첩 children 구조
- 불확실성 마킹

### 임포트 전략 (P78)
- taxon 재사용: 기존 DB의 동일 분류군 활용
- subgenus 스킵
- 신규 genera 생성
- edge cache 하이브리드 접근

## R04 분류학 입력 형식

확장된 TXT 형식 설계:

```yaml
---
reference: "Treatise 1959"
scope: comprehensive
coverage: [Trilobita]
---
```

### 마커 기반 assertion 유형
| 마커 | 의미 |
|---|---|
| (들여쓰기) | PLACED_IN (부모-자식) |
| `-` | REMOVED |
| `=>` | MOVED_TO |
| `~` | SYNONYM_OF |
| `=` | SPELLING_OF |

### 파싱 파이프라인
- YAML 헤더 파싱
- scope 선언 (comprehensive/sparse)
- 형식 확장성

## TSF 대규모 추출 (126, 130)

Treatise Source Files 14개 신규 파일, ~10,000+ genera:
- OCR 이슈 해결:
  - title case 저자명
  - word-wrapping
  - 특수 형식
- Echinodermata 등 포함
- PDF_SOURCE_STATUS.md: 28/31 TSF 파일 완료 추적

## Source-Driven Build 연동

추출된 소스 파일 → [assertion DB 구축](assertion-model.md):
- data/sources/*.txt 파일
- 6단계 빌드 파이프라인
- comprehensive removal 로직
- canonical parent_id fallback

## 관련 페이지

- [Assertion 모델](assertion-model.md) — source-driven build
- [분류학 데이터](taxonomy-data.md) — 데이터 정제
- [Packages](packages.md) — TSF 확장 패키지
- [Trilobase 개요](trilobase-overview.md)

---
*Sources: 102, 102b, 105, 109, 116, 117, 119, 120, 126, 130, 132, P78, P83, R04*
