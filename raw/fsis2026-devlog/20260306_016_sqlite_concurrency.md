# 016: SQLite 동시성 설정 개선

## 작업 내용

### `config/settings.py` DB 설정 변경

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
        'OPTIONS': {
            'timeout': 20,
            'transaction_mode': 'IMMEDIATE',
        },
    }
}
```

- `timeout: 20` — 잠금 대기 시간을 기본 5초에서 20초로 증가
- `transaction_mode: 'IMMEDIATE'` — Django 5.1 신규 옵션. write 트랜잭션 충돌을 조기 감지하여 `database is locked` 에러 감소

### WAL 모드 적용 (서버에서 1회 실행 필요)

WAL(Write-Ahead Logging) 모드는 읽기와 쓰기가 서로 블로킹하지 않아 동시성을 크게 개선한다.
DB 파일에 영구 적용되므로 한 번만 실행하면 된다.

```bash
# 서버에서 실행
sqlite3 /srv/fsis2026/db.sqlite3 "PRAGMA journal_mode=WAL;"

# 또는 Docker 컨테이너 내부에서
docker exec fsis2026 python -c "
import sqlite3
conn = sqlite3.connect('/app/db.sqlite3')
print(conn.execute('PRAGMA journal_mode=WAL;').fetchone())
conn.close()
"

# WAL 모드 확인
sqlite3 /srv/fsis2026/db.sqlite3 "PRAGMA journal_mode;"
# 출력: wal
```

## 참고

- SQLite WAL 모드 적용 시 `db.sqlite3-wal`, `db.sqlite3-shm` 파일이 함께 생성됨
- 백업 시 세 파일 모두 포함해야 함 (또는 `.backup` 명령 사용)
- 동시 사용자 5~10명 수준이면 SQLite로 충분, 그 이상은 PostgreSQL 전환 검토
