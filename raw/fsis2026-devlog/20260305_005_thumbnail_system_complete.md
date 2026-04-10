# 004: 표본 사진 썸네일 시스템 구축 완료

**작업일**: 2026-03-05
**계획 문서**: `devlog/20260305_P05_thumbnail_system.md`

## 작업 내용

### Phase 1: 인프라 구축

- `django-imagekit 6.1.0` 설치 및 `INSTALLED_APPS`에 `imagekit` 추가
- 아래 모델에 `ImageSpecField` (`thumbnail_medium`) 추가:
  - `fsis.MuseumSpecimenPhoto` (source: `data`)
  - `fsis.Figure` (source: `data`)
  - `kprdb.ReferenceTaxonSpecimen` (source: `figure_image`)
- `MuseumSpecimenModel`은 `.glb` 3D 모델 파일만 저장되어 있어 썸네일 대상에서 제외

### Phase 2: 템플릿 수정

| 템플릿 | 변경 내용 |
|--------|-----------|
| `museum_specimen_detail.html` | `width="400"` 원본 이미지 -> `thumbnail_medium.url` + `loading="lazy"` |
| `museum_specimen_form.html` | `fs.data.value.url` -> `fs.instance.thumbnail_medium.url` |
| `fig_detail.html` | 원본 이미지 -> 썸네일 + 클릭 시 원본 열기, `loading="lazy"` |
| kprdb 템플릿 | `figure_image`를 표시하는 곳 없음 (수정 불필요) |

### Phase 3: 기존 이미지 썸네일 생성

- `generate_thumbnails` 관리 커맨드 작성 및 실행
- 결과:
  - `MuseumSpecimenPhoto`: 1,930건 성공
  - `Figure`: 대상 0건
- 썸네일 캐시 위치: `uploads/CACHE/images/`

### 함께 진행된 작업

- 미사용 `Specimen`, `SpecimenPhoto` 모델 제거 (별도 devlog `003` 참조)
- `Figure` 모델에서 `specimen` ForeignKey 제거, `FigureForm`에서 `specimen` 필드 제거
- `admin.py`에서 관련 리소스/등록 코드 정리

## 썸네일 설정

```python
ImageSpecField(
    source='data',  # 또는 'figure_image'
    processors=[ResizeToFit(400, 400)],
    format='JPEG',
    options={'quality': 85}
)
```

- DB 마이그레이션 불필요 (가상 필드)
- 첫 접근 시 자동 생성, 이후 캐시에서 서빙

## 미적용 사항 (Phase 4)

- WebP 포맷 전환: 향후 검토
- `thumbnail_small` (150px): 목록 페이지에 이미지 표시 시 추가 예정
- Nginx CACHE 경로 직접 서빙 설정: 현재 Gunicorn 경유, 향후 최적화 가능
