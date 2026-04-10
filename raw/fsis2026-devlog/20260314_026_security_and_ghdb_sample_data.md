# 026: 프로덕션 보안 설정 적용 및 GHDB 샘플 데이터 입력

**작성일**: 2026-03-14

## 1. 프로덕션 보안 설정 적용 (P16)

도메인 `fsis.psok.or.kr` HTTPS 적용 완료 상태에서, P16 계획에 따라 Django 보안 설정 1단계 적용.

### 적용 내용

양쪽 `.env` (`/srv/fsis2026/.env`, `/srv/ghdb/.env`)에 추가:

```
SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True
SECURE_HSTS_SECONDS=3600
SECURE_HSTS_INCLUDE_SUBDOMAINS=True
```

### 확인 사항

- `proxy_params`에 `X-Forwarded-Proto $scheme` 이미 포함 → `SECURE_PROXY_SSL_HEADER` 정상 동작
- Nginx에서 HTTP→HTTPS 301 리다이렉트 처리 중 → `SECURE_SSL_REDIRECT=False` 유지
- HSTS 1단계(3600초) 적용 — 안정화 후 86400 → 31536000 단계적 증가 예정

### 검증 결과

```
Strict-Transport-Security: max-age=3600; includeSubDomains
Set-Cookie: csrftoken=...; Secure
```

## 2. settings_ghdb.py DATABASE_PATH 환경변수 지원

`config/settings_ghdb.py`의 DB 경로가 하드코딩되어 있어 `settings.py`와 동일하게 수정:

```python
# Before
'NAME': BASE_DIR / 'db_ghdb.sqlite3',

# After
'NAME': config('DATABASE_PATH', default=str(BASE_DIR / 'db_ghdb.sqlite3')),
```

## 3. GHDB 샘플 데이터 입력

### 화석 표본 10건 (fsis DB에서 복사)

GPS+사진 5건, 사진만 5건을 fsis DB에서 읽어 ghdb DB에 복사. `source_system=ghdb`로 설정.

| ID | 코드 | 이름 | GPS | 사진 수 |
|----|------|------|-----|---------|
| 1 | CNUNHM-F999-1 | 이매패류 (굴) | N 35°51'43.6" E 129°15'43.5" | 2 |
| 2 | CNUNHM-F999-2 | 이매패류 (굴) | N 35°51'43.6" E 129°15'43.5" | 4 |
| 3 | CNUNHM-F999-3 | 이매패류 (굴) | N 35°51'43.6" E 129°15'43.5" | 4 |
| 4 | SNUP 2563 | | 37°04'19"N 129°08'39"E | 1 |
| 5 | SNUP 2549 | | 37°04'19"N 129°08'39"E | 1 |
| 6 | KIGAM-9G340 | 조개 | | 2 |
| 7 | KIGAM-9G341 | 가리비 | | 2 |
| 8 | KIGAM-9G342 | 가리비 | | 2 |
| 9 | KIGAM-9G343 | 조개 | | 2 |
| 10 | KIGAM-9G344 | 가리비 | | 2 |

사진 파일 22장 → `/srv/ghdb/uploads/museum_specimen_photo/`에 복사.

### 암석 표본 5건 (신규 생성)

| ID | 코드 | 이름 | 분류 | 산지 | GPS |
|----|------|------|------|------|-----|
| 11 | GHDB-R001 | 화강암 | 화성암 | 전남 목포 유달산 | N 34°47' E 126°23' |
| 12 | GHDB-R002 | 편마암 | 변성암 | 경기 가평 북면 | N 37°52' E 127°30' |
| 13 | GHDB-R003 | 현무암 | 화성암 | 제주 한림읍 | N 33°23' E 126°16' |
| 14 | GHDB-R004 | 셰일 | 퇴적암 | 강원 태백 동점동 | N 37°09' E 128°59' |
| 15 | GHDB-R005 | 규암 | 변성암 | 충북 단양 매포읍 | N 36°57' E 128°22' |

암석 전용 필드(rock_category, rock_name, rock_texture, rock_structure, rock_color, rock_size, rock_description, mineral_name 등) 포함.
