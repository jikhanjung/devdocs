# P11: 백업 계획

## 현황

- **서버**: GCP VM, 디스크 50GB (사용률 90%, 여유 4.9GB)
- **운영 데이터**: `/srv/fsis2026/` (Docker 전환 준비 중)
- **소스코드**: `~/projects/fsis2026/` (git 관리)
- 디스크가 하나뿐이라 로컬 백업만으로는 불충분

## 백업 대상 분류

| 대상 | 크기 | 중요도 | 변경 빈도 | 백업 방법 |
|------|------|--------|-----------|-----------|
| db.sqlite3 | 9.8MB | 최상 | 수시 | 날짜별 스냅샷 + rsync |
| uploads/ | 8.3GB | 최상 | 가끔 | rsync 미러링 |
| .env | 62B | 높음 | 거의 없음 | rsync |
| 소스코드 | - | 높음 | 수시 | git push (이미 관리 중) |
| map_tiles/ | 550MB | 낮음 | 없음 | rsync (재생성 가능) |
| staticfiles/ | 553MB | 낮음 | 없음 | rsync (재생성 가능) |

## 백업 방식: 외부 서버에서 Pull

외부 백업 서버에서 SSH + rsync로 이 서버의 `/srv/fsis2026/`을 끌어간다.
이 서버는 SSH 접근만 허용하면 되고, 별도 설정 불필요.

### 이 서버에서 할 일

1. 백업 전용 계정 생성 (선택, 보안 강화)
   ```bash
   sudo adduser --disabled-password backupuser
   sudo usermod -aG www-data backupuser
   # /srv/fsis2026/ 읽기 권한 확인
   ```

2. SSH 키 인증 설정
   - 외부 서버의 공개키를 `~/.ssh/authorized_keys`에 등록

### 외부 백업 서버에서 할 일

1. SSH 키 생성 및 등록
   ```bash
   ssh-keygen -t ed25519 -C "backup@backupserver"
   ssh-copy-id honestjung@<운영서버IP>
   # 접속 테스트
   ssh honestjung@<운영서버IP> ls /srv/fsis2026/
   ```

2. 백업 스크립트 작성 (`/opt/backup-fsis.sh`)
   ```bash
   #!/bin/bash
   REMOTE="honestjung@<운영서버IP>"
   BACKUP_DIR="/backup/fsis2026"
   LOG="/var/log/backup-fsis.log"

   echo "$(date) - backup start" >> $LOG

   # DB는 날짜별로 별도 보관 (7일치)
   mkdir -p $BACKUP_DIR/db_history
   scp $REMOTE:/srv/fsis2026/db.sqlite3 \
       $BACKUP_DIR/db_history/db_$(date +%Y%m%d).sqlite3
   find $BACKUP_DIR/db_history -name "db_*.sqlite3" -mtime +7 -delete

   # 나머지는 미러링 (DB 제외 → 별도 처리했으므로)
   rsync -avz --delete \
     --exclude='db.sqlite3' \
     $REMOTE:/srv/fsis2026/ $BACKUP_DIR/current/

   # 현재 DB도 current에 동기화
   rsync -avz $REMOTE:/srv/fsis2026/db.sqlite3 $BACKUP_DIR/current/

   echo "$(date) - backup done" >> $LOG
   ```

3. cron 등록
   ```bash
   crontab -e
   # 매일 새벽 3시
   0 3 * * * /opt/backup-fsis.sh
   ```

### 백업 서버 디렉토리 구조

```
/backup/fsis2026/
├── current/                 # 최신 미러 (rsync --delete)
│   ├── db.sqlite3
│   ├── uploads/
│   ├── staticfiles/
│   ├── map_tiles/
│   ├── .env
│   └── logs/
└── db_history/              # DB 날짜별 스냅샷 (7일 보관)
    ├── db_20260306.sqlite3
    ├── db_20260305.sqlite3
    └── ...
```

## 복원 절차

### DB 복원
```bash
# 최신 DB
scp backupserver:/backup/fsis2026/current/db.sqlite3 /srv/fsis2026/

# 특정 날짜 DB
scp backupserver:/backup/fsis2026/db_history/db_20260305.sqlite3 /srv/fsis2026/db.sqlite3
```

### 전체 복원
```bash
rsync -avz backupserver:/backup/fsis2026/current/ /srv/fsis2026/
```

## 디스크 정리 (Docker 전환 후)

Docker 전환이 확인되면 `~/projects/fsis2026/`의 운영 데이터 원본 삭제로 약 9GB 확보:
```bash
rm -rf ~/projects/fsis2026/uploads/
rm -rf ~/projects/fsis2026/staticfiles/
rm    ~/projects/fsis2026/db.sqlite3
```

## 향후 검토

- PostgreSQL 전환 시 `pg_dump` 기반 백업으로 변경
- uploads 증가 시 증분 백업 또는 오브젝트 스토리지(GCS) 전환
- 백업 성공/실패 알림 (메일, Slack 등)
