# Multi-Package 생태계

SCODA 기반 분류학 데이터 패키지들. trilobase에서 시작하여 다른 화석 분류군으로 확장.

## 패키지 목록

| 패키지 | 대상 분류군 | 버전 | 설명 |
|---|---|---|---|
| **trilobita** (trilobase) | 삼엽충 (Trilobita) | 0.3.4 | 원본 패키지, assertion-centric |
| **brachiobase** | 완족동물 (Brachiopoda) | 0.2.4 | Treatise 기반 |
| **graptobase** | 필석류 (Graptolithina) | 0.1.0 | 초기 버전 |
| **paleocore** | 고생물학 표본/컬렉션 | 0.1.2 | 표본/산지/컬렉션 스키마 |
| **paleobase** | 통합 메타패키지 | 0.1.1 | 다중 패키지 결합 |

## PaleoCore

고생물학 표본/컬렉션/산지를 관리하는 범용 스키마:
- Phase 31-37 (2026-02-13)에서 구축
- 이중 DB 시스템: canonical Trilobase + PaleoCore
- PC_ 접두사 네임스페이싱
- PaleoCore 테이블을 Trilobase DB에서 분리 (관심사 분리)
- SCODA 메타데이터 독립 적용
- Mesozoic temporal code 중앙화 (v0.1.2)

## Brachiobase

완족동물 분류학 데이터:
- Treatise 기반 분류 체계
- tree chart + profile comparison compound view
- Mesozoic temporal code 추가 (v0.2.4): Triassic, Jurassic, Cretaceous
- P85: 공통 분류학 템플릿을 scoda_engine_core 모듈로 추출

## Graptobase

필석류 분류학 데이터:
- v0.1.0 초기 버전 (2026-03-14)

## Paleobase (메타패키지)

여러 분류학 패키지를 결합하는 메타패키지:

### 설계 옵션 검토 (P88)
- Bundle (단일 DB) — 복잡
- **Metapackage (의존성 선언)** — 채택된 접근
- Single-DB (병합) — 비현실적

### 구현
- Stage 0/1 (134): 초기 구축
- bindings 수정 (135): root taxon명/랭크 매칭 (CHELICERATA/Subphylum 등)
- meta-tree 구조 (137, 139): 상위 계층 연결
- composite tree API

### 패키지 이름 변경 (133)
- taxonomy 패키지 → taxon_names로 리네이밍

## Timeline 기능

- P87: timeline SCODA 패키지 설정
- 124: MYA 슬라이더 구현
- 125: brachiobase에 timeline 적용
- ICS 지질연대 데이터 연동

## Treatise Source Files (TSF) 확장 (126, 130)

- 14개 신규 소스 파일, ~10,000+ genera 추출
- OCR 이슈 해결: title case 저자, word-wrapping, 특수 형식
- 28/31 TSF 파일 완료
- Echinodermata 포함 대규모 추출

## 다중 패키지 서빙 (P86)

- 단일 scoda-engine 인스턴스에서 여러 패키지 동시 서빙
- 패키지 선택 UI

## 버전 일괄 업데이트 (136)

7개 패키지 동시 패치 버전 범프:
- trilobita 0.3.3 → 0.3.4
- paleobase 0.1.0 → 0.1.1
- 기타 패키지들

## 관련 페이지

- [SCODA 아키텍처](scoda.md) — 패키지 규격
- [Trilobase 개요](trilobase-overview.md) — 원본 프로젝트
- [Treatise 추출](treatise-extraction.md) — 소스 데이터 추출
- [시각화](visualization.md) — 트리/차트/비교
- [PaperMeister 아키텍처](papermeister-architecture.md) — domain DB 후보가 PaperMeister corpus에서 파생되는 파이프라인
- [Noematica 브랜드](noematica-brand.md) — domain knowledge 레이어

---
*Sources: 039-046, 125, 126-130, 132-139, P27-P37, P85-P89, archive phase 31-43*
