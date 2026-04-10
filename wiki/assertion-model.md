# Assertion-Centric 데이터 모델

다중 분류 의견(taxonomic opinions)을 체계적으로 관리하기 위한 assertion 중심 데이터 모델. v0.3.0에서 trilobase의 기본 모델로 통합되었다.

## 핵심 개념

기존 모델에서는 `parent_id`가 계층 구조를 직접 저장했지만, assertion 모델에서는 **모든 분류 관계가 assertion(주장)으로 표현**되고, 계층 구조는 이로부터 파생된다.

## 스키마

| 테이블 | 역할 |
|---|---|
| **taxon** | 분류군 엔티티 (이름, 랭크, 저자) |
| **assertion** | 분류학적 주장 (PLACED_IN, SYNONYM_OF, SPELLING_OF) |
| **reference** | 참고문헌 출처 |
| **classification_profile** | 분류 프로필 (특정 관점의 분류 체계) |
| **classification_edge_cache** | 프로필별 계산된 부모-자식 관계 캐시 |

## Assertion 유형 (opinion_type)

| 유형 | 의미 | 예시 |
|---|---|---|
| **PLACED_IN** | 상위 분류군 배치 | "Genus X는 Family Y에 속한다" |
| **SYNONYM_OF** | 동의어 관계 | "Genus A는 Genus B의 동의어다" |
| **SPELLING_OF** | 철자 변형 | "Dokimocephalidae = Dokimocephalinae" |

## assertion_status 분류

| 상태 | 의미 |
|---|---|
| asserted | 명시적 주장 |
| incertae_sedis | 불확실한 분류 |
| indet | 미결정 |
| questionable | 의문 |

## Classification Profiles

특정 참고문헌/관점에 따른 분류 체계:

| 프로필 | 설명 |
|---|---|
| **JA2002** | Jell & Adrain (2002) — 정전 분류 |
| **treatise1959** | Treatise on Invertebrate Paleontology (1959) — 독립 구축 |
| **treatise1997** | Treatise (1997, 기존 2004로 오표기) — treatise1959 기반 확장 |

### 프로필 구축 방식
- 독립(standalone): treatise1959 — 자체 소스로부터 완전 구축
- 연쇄(chained): treatise1997 — treatise1959를 기반으로 변경사항만 추가

### edge_cache
- 프로필별로 계산된 부모-자식 관계 저장
- 프로필 전환 시 캐시 참조로 트리 즉시 재구축
- genera_count: 정적 컬럼 제거 → edge_cache 서브쿼리로 동적 계산

## 동의어 → 의견 마이그레이션

1,055건 동의어 레코드를 taxonomic_opinions (후에 assertions)로 마이그레이션:
- synonyms 테이블 → VIEW로 전환 (하위 호환)
- synonym_type 보존: j.s.s., j.o.s., preoccupied, replacement, suppressed
- taxon_bibliography.synonym_id → opinion_id 리네이밍
- fide→bibliography 매칭: 433 → 566건 (개선된 매칭 알고리즘)
- is_accepted 우선순위 로직

## Assertion 생성 경로

parent_id에서 assertion 폭발(explosion):
- 5,003 PLACED_IN assertions (parent_id로부터 자동 생성)
- 1,139 opinions (기존 taxonomic_opinions에서 변환)
- 총 6,142+ assertions → v0.2.0에서 8,331 assertions

## Comprehensive Scope & Removal (R03)

- **comprehensive scope**: 참고문헌이 특정 범위의 모든 분류군을 다루는 경우
- **sparse scope**: 일부만 언급하는 경우
- comprehensive scope + 미언급 = 의도적 제거(removal)
- reference_scope 테이블 설계
- removed taxa를 diff view에서 별도 패널로 표시

## Source-Driven Build (P83)

소스 파일 기반 assertion DB 구축:
- `data/sources/*.txt` 파일에서 파싱
- [R04 형식](treatise-extraction.md): YAML 헤더 + 마커 기반 assertion
- 6단계 파이프라인: 파싱 → taxon 생성 → assertion 생성 → 프로필 구축 → edge_cache → 검증
- canonical parent_id fallback

## v0.3.0 통합 (2026-03-11)

- assertion-centric DB가 기본 trilobase로 승격
- is_accepted 컬럼 제거
- 스크립트 이름 통합, 60+ 레거시 스크립트 아카이브
- artifact_id 변경

## CRUD (P80)

매니페스트의 editable_entities 선언 기반:
- taxon, assertion, reference, classification_profile 편집 가능
- FK autocomplete
- linked_table 인라인 편집
- admin 모드

## 관련 페이지

- [Trilobase 개요](trilobase-overview.md)
- [분류학 데이터](taxonomy-data.md) — 데이터 품질 및 rebuild 파이프라인
- [시각화](visualization.md) — 프로필 비교 시각화
- [Treatise 추출](treatise-extraction.md) — 소스 데이터 형식

---
*Sources: 077, 084, 086-088, 096-097, 101, 107-108, 110, 118-123, P50-P53, P60, P70, P74, P74b, P80, P83, R01, R03, R04, R05*
