# 분류학 데이터 (정제 및 파이프라인)

삼엽충 분류학 데이터의 정제, 검증, 재구축 과정.

## 원본 데이터 소스

| 소스 | 내용 |
|---|---|
| **trilobite_genus_list.txt** | Jell & Adrain (2002) 삼엽충 속 목록 (5,113 genera) |
| **adrain2011.txt** | Adrain (2011) 분류 계층 (Order~Superfamily) |
| **Treatise 1959** | Treatise on Invertebrate Paleontology Part O (1959) |
| **Treatise 1997** | Treatise Part O Revised (실제 1997, 기존 2004로 오표기) |

## Phase 1-7: 초기 데이터 정제 (2026-02-04)

### 라인 정규화 (Phase 1)
- 소프트 하이픈 1,357개 제거
- 가비지 라인 제거, 연속 라인 병합
- 5,095 → 5,105 라인

### 문자 수정 (Phase 2)
- 체코 저자명 복원: ŠNAJDR, PŘIBYL, VANĚK
- 403건 수정 (369 + 31 + 3)

### 데이터 검증 (Phase 3)
- 형식 검증, 과명 검증, 시간 코드 확인
- 중복 29건 식별, 괄호 균형 검사

### DB 생성 (Phase 4)
- SQLite 스키마: 5,113 taxa, 191 families, 1,055 synonyms

### 정규화 (Phase 5-6)
- 동의어 파싱 향상: 90.5% → 97.7%
- 산지/국가 관계 테이블 생성
- 과명 소프트 하이픈/제어 문자 정리

### 계층 통합 (Phase 7)
- adrain2011.txt에서 Order/Suborder/Superfamily 통합
- self-referential taxonomic_ranks 테이블

## Phase 8-10: 통합 (2026-02-05)

- taxonomic_ranks + families + taxa → 단일 테이블 통합 (5,338 레코드)
- genus_formations (4,854), genus_locations (4,841) 다대다 관계 테이블
- 참고문헌 파싱: 2,130 articles/books/chapters

## 후속 데이터 품질 작업 (2026-02-19~28)

### parent_id NULL 해결
- 13 genera: raw_entry에서 가족 매핑 복원 (세미콜론/콜론 혼동, 비표준 형식)
- 25 families → "Order Uncertain" (id=144) 배치
- 68 genera: taxonomic_opinions으로 incertae_sedis/indet/questionable 상태 부여
- 최종: valid genus parent_id NULL = 0

### 동의어 링크 수정
- 24건 미연결 → 18 정확 매칭 + 4 철자 수정 + 1 ICZN Opinion 1259
- 연결률: 97.6% → 99.9% (1,054/1,055)

### 인코딩/OCR 수정
- BRAÑA 인코딩 오류 13건 (° → ñ)
- PDF 줄바꿈 하이픈 165건 제거
- 저자 공백 44건 수정
- 제어 문자 2건 제거

### taxon_bibliography 접합 테이블
- LIKE 기반 → FK 기반 매칭
- 4,040 original_description + 433 fide 링크
- 저자 성 추출, 연도 접미사 구분, 신뢰도 레벨
- fide 매칭 개선: 433 → 566건

### 지리 데이터 수정 (098)
- country_id: 3,769/4,841 수정 (77.8%), China→England 오류 1,023건 해소
- formation 정렬: 350건 (Type 1/2/3 분류)
- region_id: 272건, 93 신규 지역 생성

### Agnostida Order 생성
- 신규 Order: 10 families, 162 genera
- PLACED_IN 의견 + SPELLING_OF 의견 유형 추가
- Order Uncertain: 81 → 68 families

### temporal_code NULL 채우기
- 84건 raw_entry 패턴 매칭으로 충전

## Rebuild 파이프라인 (099)

소스 텍스트에서 DB를 완전 재구축하는 8단계 모듈식 파이프라인:

```
1. Text Cleaning    → 라인 정규화, 문자 수정
2. Hierarchy Parse  → adrain2011.txt 파싱
3. Genus Parse      → trilobite_genus_list.txt 파싱 (개선된 산지/국가 분리)
4. Schema Load      → SQLite 스키마 생성
5. PaleoCore Gen    → PaleoCore DB 독립 생성
6. Junction Tables  → 산지/국가/참고문헌 접합 테이블
7. SCODA Metadata   → artifact_metadata, provenance, queries, manifest
8. Validation       → 35/35 검증 체크
```

### Rebuild Diff 해소 (100)
35/35 검증 체크를 통과하기 위한 반복적 수정:
- 계층 이름 파싱 (3건)
- SPELLING_OF 과명 매핑 (47건)
- INDET parent_id (14건)
- 동의어 senior_name 추출 (9건, 6 패턴)
- 동의어 의견 중복 (4건)
- Type 3 formation 재분류 (356건)
- 괄호 내용 처리 (5건)

### 재사용 가능 컴포넌트
- text cleaning 프레임워크
- country normalization 맵
- formation whitelist
- bibliography matching

## 관련 페이지

- [Trilobase 개요](trilobase-overview.md) — 전체 프로젝트
- [Assertion 모델](assertion-model.md) — assertion 기반 데이터 관리
- [Treatise 추출](treatise-extraction.md) — 추가 소스 데이터

---
*Sources: archive/001-014, 078-082, 083, 084, 086-088, 092-093, 096-100, P01-P06, P62-P64, P66, P71-P73*
