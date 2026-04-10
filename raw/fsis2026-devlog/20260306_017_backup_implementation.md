# 017: DB 백업 자동화 구현

**날짜**: 2026-03-06

## 개요

운영서버(GCP 34.64.158.160)의 데이터를 개발서버(m710q)로 pull하여 백업하는 시스템 구축.
로컬 디스크와 NAS 이중 백업, 계층형 보관 정책 적용.

## 백업 스크립트

### 1. FSIS2026 백업 (`backup-fsis.sh`)

- **위치**: `/home/jikhanjung/scripts/backup-fsis.sh`
- **cron**: `0 3 * * *` (매일 새벽 3시)
- **방식**: SSH + scp/rsync로 운영서버에서 pull

#### 백업 대상

| 대상 | 방식 | 비고 |
|------|------|------|
| db.sqlite3 + WAL/SHM | scp 날짜별 스냅샷 | WAL 모드 3파일 모두 포함 |
| uploads/ | rsync --delete 미러링 | 매일 최신 상태 동기화 |
| .env | scp 복사 | 설정 파일 |

#### uploads 스냅샷 (월별/연별)

| 시점 | 유형 | 설명 |
|------|------|------|
| 매달 1일 | diff | 직전 full 대비 변경분만 (`--compare-dest`) |
| 1월 1일 | full | 연간 전체 백업 (새 기준점) |
| 수동 `--full-snapshot` | full | 즉시 전체 백업 생성 |

### 2. DolfinServer 백업 (`database_backup.sh`)

- **위치**: `/home/jikhanjung/scripts/database_backup.sh`
- **cron**: `0 1 * * *` (매일 새벽 1시)
- **방식**: 로컬 cp (같은 서버 내 DB)

## 계층형 DB 보관 정책

두 스크립트 모두 동일한 정책 적용:

```
┌─────────────────────────────────────────────────────┐
│  90일 이내          매일 보관                        │
│  90일 초과          매달 1일만 보관 (나머지 삭제)     │
│  12월 1일           영구 보관 (연간 아카이브)         │
└─────────────────────────────────────────────────────┘
```

- **로컬**: 30일 매일 → 이후 월초만
- **NAS**: 90일 매일 → 이후 월초만
- **12월 1일**: 로컬/NAS 모두 영구 보관

## 저장 경로

### FSIS2026

```
로컬: /home/jikhanjung/backups/fsis2026/
├── current/                    # 최신 미러
│   ├── db.sqlite3 (+wal, +shm)
│   ├── uploads/
│   └── .env
├── db_history/                 # DB 날짜별 스냅샷
│   ├── db_20260306.sqlite3 (+wal, +shm)
│   └── ...
├── uploads_snapshots/          # uploads 월별/연별 스냅샷
│   ├── 2026_full/              # 연간 전체 백업
│   ├── 202602_diff/            # 월별 변경분
│   └── ...
└── backup.log

NAS: /nas/JikhanJung/fsis2026_backup/
├── current/                    # (동일 구조)
├── db_history/
└── uploads_snapshots/
```

### DolfinServer

```
로컬: /home/jikhanjung/backup/
├── db.sqlite3.YYYY-MM-DD      # 날짜별
└── backup.log

NAS: /nas/JikhanJung/dolfinid_backup/
└── db.sqlite3.YYYY-MM-DD
```

## 사전 설정

### SSH 키

- 개발서버 → 운영서버 SSH 키 인증 설정 완료
- 키: `/home/jikhanjung/.ssh/id_ed25519` (jikhanjung@m710q)
- 운영서버 `~/.ssh/authorized_keys`에 등록됨

### crontab

```
0 1 * * * /home/jikhanjung/scripts/database_backup.sh
0 3 * * * /home/jikhanjung/scripts/backup-fsis.sh
```

## 수동 실행

```bash
# FSIS 일반 백업
/home/jikhanjung/scripts/backup-fsis.sh

# FSIS uploads 전체 백업 (초기 기준점 생성 등)
/home/jikhanjung/scripts/backup-fsis.sh --full-snapshot

# DolfinServer 백업
/home/jikhanjung/scripts/database_backup.sh
```

## 용량 예측

### FSIS2026 (DB 9.8MB, uploads 8.3GB)

| 항목 | 용량 |
|------|------|
| current/ (미러) | 8.3GB |
| DB 30일분 (로컬) | ~300MB |
| DB 90일분 (NAS) | ~900MB |
| uploads full 1회 | 8.3GB |
| uploads diff 월별 | 수~수백MB (변경량에 따라) |

### DolfinServer (DB 927MB)

| 항목 | 용량 |
|------|------|
| 로컬 30일분 | ~28GB |
| NAS 90일분 | ~83GB |
| 월초 보관 (12개) | ~11GB |

## 복원 절차

### FSIS DB 복원

```bash
# 최신 DB
scp m710q:/home/jikhanjung/backups/fsis2026/current/db.sqlite3* /srv/fsis2026/

# 특정 날짜 DB
scp m710q:/home/jikhanjung/backups/fsis2026/db_history/db_20260306.sqlite3* /srv/fsis2026/
# 파일명을 db.sqlite3, db.sqlite3-wal, db.sqlite3-shm 으로 변경
```

### FSIS uploads 복원

```bash
# 최신 미러에서 복원
rsync -avz m710q:/home/jikhanjung/backups/fsis2026/current/uploads/ /srv/fsis2026/uploads/

# full 스냅샷 + diff 조합 복원
rsync -avz m710q:.../uploads_snapshots/2026_full/ /srv/fsis2026/uploads/
rsync -avz m710q:.../uploads_snapshots/202603_diff/ /srv/fsis2026/uploads/
```
