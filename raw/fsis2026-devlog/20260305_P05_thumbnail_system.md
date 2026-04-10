# P05: 표본 사진 썸네일 시스템 구축 계획

## 현황 분석

### 문제점
- 표본정보 페이지(목록/상세)에서 원본 해상도 이미지를 그대로 로드
- HTML `width="400"` 속성으로만 크기 축소 — 실제 전송 데이터는 원본 그대로
- `uploads/museum_specimen_photo/`에 1,819개 파일 존재
- 네트워크 대역폭 낭비, 페이지 로딩 속도 저하

### 대상 모델 (이미지 필드 보유)

| 모델 | 앱 | 이미지 필드 | upload_to | 비고 |
|------|-----|------------|-----------|------|
| MuseumSpecimenPhoto | fsis | `data` | `museum_specimen_photo` | 주요 대상 (1,819건) |
| MuseumSpecimenModel | fsis | `data` | `museum_specimen_model` | 3D 모델 이미지 |
| SpecimenPhoto | fsis | `data` | `specimen_photo` | |
| Figure | fsis | `data` | `figure` | 참고문헌 도판 |
| ReferenceTaxonSpecimen | kprdb | `figure_image` | `reference_figures` | kprdb 표본 도판 |

### 이미지 사용 위치

- `museum_specimen_detail.html`: 상세보기에서 `width="400"`으로 표시, 클릭 시 원본 열기
- `museum_specimen_form.html`: 편집 폼에서 `width="400"`으로 미리보기
- `museum_specimen_list.html`: 목록에서는 `get_photos_count()`만 표시 (이미지 미표시)
- `fig_detail.html`: Figure 이미지 표시
- kprdb 쪽 참고문헌 상세에서 도판 이미지 표시

---

## 구현 계획

### 접근 방식: django-imagekit 기반 썸네일 자동 생성

django-imagekit(또는 Pillow 직접 사용)으로 이미지 업로드 시 썸네일을 자동 생성하고,
기존 이미지에 대해서는 관리 커맨드로 일괄 생성한다.

### Phase 1: 썸네일 인프라 구축

#### 1-1. 패키지 설치
```bash
pip install django-imagekit
# Pillow는 이미 설치됨
```

#### 1-2. settings.py 설정
```python
INSTALLED_APPS = [
    ...
    'imagekit',
    ...
]
```

#### 1-3. 모델 수정 — ImageSpecField 추가

**방안 A: django-imagekit의 ImageSpecField 사용 (권장)**
```python
from imagekit.models import ImageSpecField
from imagekit.processors import ResizeToFit, ResizeToFill

class MuseumSpecimenPhoto(models.Model):
    data = models.ImageField(upload_to="museum_specimen_photo", blank=True)

    # 목록/폼 미리보기용 (400px)
    thumbnail_medium = ImageSpecField(
        source='data',
        processors=[ResizeToFit(400, 400)],
        format='JPEG',
        options={'quality': 85}
    )

    # 향후 목록 페이지 등에서 작은 썸네일 필요시 (150px)
    thumbnail_small = ImageSpecField(
        source='data',
        processors=[ResizeToFit(150, 150)],
        format='JPEG',
        options={'quality': 80}
    )
    ...
```

- `ImageSpecField`는 DB 컬럼을 추가하지 않음 (마이그레이션 불필요)
- 첫 접근 시 자동 생성, 이후 캐시에서 서빙
- 캐시 디렉토리: `IMAGEKIT_DEFAULT_CACHEFILE_BACKEND` 설정 가능

**방안 B: Pillow 직접 사용 + thumbnail 필드 추가**
```python
class MuseumSpecimenPhoto(models.Model):
    data = models.ImageField(upload_to="museum_specimen_photo", blank=True)
    thumbnail = models.ImageField(upload_to="museum_specimen_photo/thumbs", blank=True)

    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        if self.data and not self.thumbnail:
            self._generate_thumbnail()
```

- DB 마이그레이션 필요
- 더 세밀한 제어 가능하지만 관리 코드가 늘어남

**결론: 방안 A (django-imagekit) 권장**
- 마이그레이션 불필요, 기존 데이터에 영향 없음
- 지연 생성(lazy) 또는 사전 생성(eager) 선택 가능
- Django 커뮤니티에서 검증된 라이브러리

#### 1-4. 동일하게 적용할 모델 목록

```python
# fsis/models.py
class MuseumSpecimenPhoto  → thumbnail_medium, thumbnail_small
class MuseumSpecimenModel  → thumbnail_medium
class SpecimenPhoto        → thumbnail_medium, thumbnail_small
class Figure               → thumbnail_medium

# kprdb/models.py
class ReferenceTaxonSpecimen → thumbnail_medium (source='figure_image')
```

### Phase 2: 템플릿 수정

#### 2-1. 상세 페이지 (museum_specimen_detail.html)

변경 전:
```html
<a href="/media/{{ specimen_photo.data }}" target="_blank">
    <img width="400" src="/media/{{ specimen_photo.data }}"/>
</a>
```

변경 후:
```html
<a href="/media/{{ specimen_photo.data }}" target="_blank">
    <img src="{{ specimen_photo.thumbnail_medium.url }}" alt="{{ specimen_photo.name }}"/>
</a>
```

- 썸네일을 표시하되, 클릭하면 원본 이미지를 새 탭에서 열기 (기존 동작 유지)

#### 2-2. 편집 폼 (museum_specimen_form.html)

변경 전:
```html
{% if fs.data.value %}<img src="{{fs.data.value.url}}" width="400"/>{% endif %}
```

변경 후:
```html
{% if fs.instance.data %}<img src="{{ fs.instance.thumbnail_medium.url }}"/>{% endif %}
```

#### 2-3. 기타 템플릿
- `fig_detail.html`: Figure 이미지에 동일 적용
- kprdb 참고문헌 상세: ReferenceTaxonSpecimen 도판에 적용

### Phase 3: 기존 이미지 썸네일 사전 생성

#### 3-1. 관리 커맨드 작성

```bash
python manage.py generate_thumbnails
```

- 기존 1,819개 + 기타 이미지에 대해 썸네일 일괄 생성
- imagekit의 `generateimages` 커맨드 활용 가능:
  ```bash
  python manage.py generateimages
  ```
- 또는 커스텀 커맨드로 진행률 표시 포함 구현

#### 3-2. 캐시 디렉토리 구조

```
uploads/
  CACHE/                          # imagekit 기본 캐시 디렉토리
    images/
      museum_specimen_photo/
        {hash}.jpg                # 자동 생성된 썸네일
  museum_specimen_photo/          # 원본 (기존 그대로)
  museum_specimen_model/          # 원본
  ...
```

### Phase 4: 추가 최적화 (선택)

#### 4-1. 이미지 Lazy Loading
```html
<img src="{{ photo.thumbnail_medium.url }}" loading="lazy" alt="..."/>
```

#### 4-2. WebP 포맷 지원
```python
thumbnail_medium = ImageSpecField(
    source='data',
    processors=[ResizeToFit(400, 400)],
    format='WEBP',
    options={'quality': 80}
)
```
- WebP는 JPEG 대비 25-35% 용량 절감
- 브라우저 호환성 확인 필요 (IE11 미지원, 그 외 대부분 지원)

#### 4-3. Nginx에서 정적 파일 직접 서빙
- 현재 Nginx+Gunicorn 구조이므로 `/media/CACHE/` 경로도 Nginx에서 직접 서빙하도록 설정

---

## 작업 순서 및 예상 영향

| 단계 | 작업 | DB 마이그레이션 | 서비스 중단 |
|------|------|:---:|:---:|
| 1-1 | django-imagekit 설치 | X | X |
| 1-2 | settings.py 수정 | X | X |
| 1-3 | 모델에 ImageSpecField 추가 | X | X |
| 2 | 템플릿 수정 | X | X |
| 3 | 기존 이미지 썸네일 생성 | X | X |
| 4 | 추가 최적화 | X | X |

- **DB 마이그레이션 불필요**: ImageSpecField는 가상 필드
- **서비스 중단 불필요**: 썸네일이 없으면 첫 접근 시 자동 생성
- **롤백 용이**: ImageSpecField 제거하고 템플릿 원복하면 끝

## 기대 효과

- 원본 이미지 평균 2-5MB → 썸네일 약 30-80KB (약 95% 용량 감소)
- 표본 상세 페이지 로딩 시간 대폭 단축
- 서버 대역폭 절감
