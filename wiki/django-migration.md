# Django 마이그레이션 (nkfadmin → fsis2026)

기존 nkfadmin 프로젝트(Django 3.1.2)를 새로운 fsis2026 프로젝트(Django 5.x)로 마이그레이션한 과정.

## 마이그레이션 개요

| 항목 | 이전 | 이후 |
|---|---|---|
| 프로젝트명 | nkfadmin | fsis2026 |
| Django 버전 | 3.1.2 → 5.1 → **5.2 LTS** |
| Python | — | 3.10+ (3.12 Docker) |
| 앱명 | nkfadmin | fsis |
| DB | SQLite | SQLite (WAL 모드) |

## 단계별 업그레이드 경로

```
Django 3.1.2 → 3.2 LTS → 4.2 LTS → 5.1 → 5.2 LTS
```

### 주요 변경사항
- USE_L10N 폐기
- DEFAULT_AUTO_FIELD 설정 필요
- on_delete 이미 명시 (호환성 양호)
- path() 이미 사용 중 (호환성 양호)

### Django 5.2 LTS 업그레이드 (2026-03-10)
- 5.1.15 → 5.2.12 LTS (보안 지원 2028년까지)
- 서드파티 호환 확인: simple_history, import_export, autocomplete-light, bootstrap5
- 마이그레이션 적용, 전 앱 테스트 통과

## 앱 마이그레이션 (nkfadmin → fsis)

7단계 작업:
1. django-admin startproject fsis2026
2. nkfadmin → fsis 앱 복사
3. 전체 이름 변경 (템플릿 참조 포함)
4. kprdb 앱 조정
5. settings.py 구성
6. .gitignore 설정
7. 검증

### 데이터 마이그레이션
- 37,232행 SQLite 데이터 이전
- 초기 마이그레이션 생성 + 실행
- 160개 파일 변경

## 패키지 호환성

| 패키지 | 상태 |
|---|---|
| django-simple-history | 호환 |
| django-import-export | 호환 |
| django-autocomplete-light | 호환 |
| django-multiselectfield | 확인 필요 |
| django-archive | 확인 필요 |
| django-imagekit 6.1.0 | 신규 추가 |
| python-decouple | 신규 추가 |

## 코드 정리

- 미사용 Specimen, SpecimenPhoto 모델 삭제 (0건 데이터)
- Figure 모델에서 specimen FK 제거
- nkfadmin 파일명/클래스명 → fsis_ 접두사 변경

## 관련 페이지

- [배포 인프라](deployment.md) — Docker 기반 배포
- [프로젝트 개요](fsis2026-overview.md)

---
*Sources: 001, 004, 025, P01, P02, P03*
