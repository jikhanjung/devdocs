# 003: 미사용 Specimen/SpecimenPhoto 모델 정리

## 작업 일자
2026-03-05

## 배경
썸네일 시스템 구축(P05) 준비 중 Specimen, SpecimenPhoto 모델이 사용되지 않음을 확인.
- 뷰/URL/템플릿에서 참조 없음
- DB 데이터 0건 (Specimen: 0, SpecimenPhoto: 0, Figure with specimen FK: 0)
- admin import-export 등록만 존재

## 삭제 대상
- `Specimen` 모델 (fsis/models.py)
- `SpecimenPhoto` 모델 (fsis/models.py)
- `Figure.specimen` FK 필드 (nullable, 참조 데이터 0건)
- admin.py: SpecimenResource, SpecimenAdmin, SpecimenPhotoResource, SpecimenPhotoAdmin 및 등록
- forms.py: FigureForm에서 specimen 필드 제거
- fig_detail.html: figure.specimen 분기 제거

## 마이그레이션
- `fsis/migrations/0002_remove_specimen_location_remove_specimen_reference_and_more.py`
- Django check 통과, migrate 적용 완료

## 영향
- Figure 모델은 유지 (specimen FK만 제거)
- 기존 기능에 영향 없음
