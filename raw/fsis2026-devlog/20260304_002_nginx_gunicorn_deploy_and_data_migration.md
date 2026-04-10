# Nginx + Gunicorn 배포 및 데이터 마이그레이션

**날짜**: 2026-03-04
**서버**: 34.64.158.160 (dolfinid-2, 운영 서버에서 직접 작업)

## 1. Nginx + Gunicorn 배포 설정

### 변경 사항
- Apache2 중지 및 비활성화
- Gunicorn 25.1.0 설치 (`requirements.txt` 갱신)
- `config/settings.py`: `ALLOWED_HOSTS = ['*']`
- `python manage.py collectstatic` 실행

### 생성한 설정 파일
| 파일 | 설명 |
|------|------|
| `/etc/systemd/system/gunicorn-fsis.socket` | Unix 소켓 `/run/gunicorn-fsis.sock` |
| `/etc/systemd/system/gunicorn-fsis.service` | Gunicorn 3 workers, User=honestjung |
| `/etc/nginx/sites-available/fsis2026` | 포트 80, static/media alias, proxy_pass |

### Nginx 설정 요약
- `/static/` → `/home/honestjung/projects/fsis2026/staticfiles/`
- `/media/` → `/home/honestjung/projects/fsis2026/uploads/`
- `/` → `proxy_pass http://unix:/run/gunicorn-fsis.sock`

## 2. 구 DB → 신규 DB 데이터 마이그레이션

### 원본
- `nkfsite.db.sqlite3` (구 nkfsite Django 3.1.2 프로젝트 DB)

### 결과
- **37,232 rows** 이전 완료
- 테이블명 매핑: `nkfadmin_*` → `fsis_*`, `kprdb_*`는 동일
- 스키마 완전 일치, 컬럼 변경 없음

### 주요 데이터
| 테이블 | 건수 |
|--------|------|
| auth_user | 42 |
| fsis_museumspecimen | 817 |
| fsis_museumspecimenphoto | 1,930 |
| fsis_historicalmuseumspecimen | 3,894 |
| kprdb_reference | 871 |
| kprdb_author | 842 |
| kprdb_referenceauthor | 2,535 |
| kprdb_useractivity | 14,548 |
| 기타 kprdb + history | ~12,000 |

### 백업
- `db.sqlite3.backup` — 마이그레이션 전 빈 DB 백업

## 3. 업로드 파일 이동

### 원본 경로
- `/home/honestjung/from_dolfinid-1/nkfsite/uploads/`

### 이동 결과
| 항목 | 원본 경로 | 대상 경로 | 건수 |
|------|-----------|-----------|------|
| 표본 사진 | `museum_specimen_photo/` | `uploads/museum_specimen_photo/` | 1,930 |
| 문헌 PDF | `references/` | `uploads/references/` | 326 |

- DB의 파일 경로(`fsis_museumspecimenphoto.data`, `kprdb_reference.data`)가 MEDIA_ROOT 기준 상대경로라서 DB 수정 불필요

## 4. 템플릿 오류 수정

### 문제
HTML 주석(`<!-- -->`) 안에 있는 `{% url %}` 태그도 Django 템플릿 엔진이 처리하여 `NoReverseMatch` 에러 발생

### 수정
`<!-- -->` → `{% comment %}...{% endcomment %}` 변환

| 파일 | 수정 내용 |
|------|-----------|
| `fsis/templates/fsis/museum_specimen_list.html` | `reference_add2` URL |
| `fsis/templates/fsisnavbar.html` | `fsis_user_login`, `user_register` URL |
| `fsis/templates/base.html` | `chrono_chart`, `ref_list` URL |
| `fsis/templates/fsis/chrono_list.html` | `chrono_detail` URL |
| `kprdb/templates/kprdb/user_detail_admin.html` | `user_change_password_admin` URL |
| `kprdb/templates/kprdb/reference_detail.html` | `reference_edit`, `popup_specimen_edit` URL |
| `kprdb/templates/kprdb/reference_list.html` | `reference_list` 정렬, `reference_add2` URL |

## 5. 지리정보(화석지도) 페이지 마이그레이션

### 이전 상태
- `http://34.64.158.160/nkfadmin/nkfadmin_kakao.html` (구 Apache 정적 파일, 사라짐)
- 마커 데이터: `nkfadmin_data.js`에 JSON 하드코딩 (60건)
- 마커 링크: `http://34.64.158.160:8000/nkfadmin/...` 하드코딩

### 변경 후
- **URL**: `/fsis/fossil_map/` (Django 뷰)
- **마커 데이터**: `/fsis/fossil_map_data/` JSON API로 DB에서 동적 로드 (92건)
- **마커 링크**: `/fsis/museum_specimen_detail/{id}/`
- **맵타일**: `{% static 'fsis/map_tiles' %}` (550MB, .gitignore에 포함)

### 추가/수정 파일
| 파일 | 설명 |
|------|------|
| `fsis/views.py` | `fossil_map`, `fossil_map_data` 뷰 추가 |
| `fsis/urls.py` | `fossil_map/`, `fossil_map_data/` URL 추가 |
| `fsis/templates/fsis/fossil_map.html` | 카카오맵 템플릿 (fetch API 사용) |
| `fsis/static/fsis/nkfadmin_occurrence.js` | InfoWindow HTML 문자열 방식으로 재작성 |
| `fsis/static/fsis/jquery-3.5.1.min.js` | jQuery (카카오맵 페이지용) |
| `fsis/static/fsis/map_tiles/` | 지질도 타일 54,610개 (git 제외) |

### 기술 변경 포인트
- Kakao Maps InfoWindow: DOM element 방식 → HTML 문자열 방식 (DOM 참조 갱신 문제 해결)
- `nkfadmin_data.js` 정적 파일 → `fossil_map_data` Django 뷰에서 `MuseumSpecimen.get_decimal_latitude_longitude()` 호출하여 동적 생성

## 참고

### 서비스 관리 명령
```bash
sudo systemctl restart gunicorn-fsis
sudo systemctl reload nginx
sudo systemctl status gunicorn-fsis
sudo systemctl status nginx
```

### git에 포함되지 않는 파일
- `db.sqlite3` — SQLite DB
- `uploads/` — 미디어 파일 (사진, PDF)
- `fsis/static/fsis/map_tiles/` — 지질도 타일 (550MB)
- `staticfiles/` — collectstatic 결과
- `map_tiles.zip` — 타일 원본 압축 파일
