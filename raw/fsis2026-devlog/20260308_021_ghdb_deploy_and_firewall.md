# ghdb 배포 및 방화벽 트러블슈팅

**날짜**: 2026-03-08

## 작업 내용

### ghdb navbar에 시스템관리 메뉴 추가

**커밋**: `eb1b67d`

`is_staff=True`인 사용자에게 시스템관리 드롭다운 표시:
- Django Admin (`/admin/`)
- 사용자 관리 (`/admin/auth/user/`)

fsis는 그룹(`Professors`) 기반이지만, ghdb는 `user_obj.is_staff`로 간단하게 처리.

### Docker 0.2.0 → 0.2.1

**커밋**: `e7c2ab1`

- 0.2.0: ghdb 앱 + 암석 표본 필드 + source_system (entrypoint.sh 변경 반영 전 빌드)
- rebase 후 entrypoint.sh 변경(worker count 설정) 반영하여 0.2.0 재빌드/push
- 0.2.1: navbar admin 메뉴 추가

## ghdb 서버 배포

### 서버 구성 (34.64.158.160)

```
Nginx (:80)  → fsis 컨테이너 (:8000)
Nginx (:8001) → ghdb 컨테이너 (:8002)
Nginx (:8080) → scoda 컨테이너 (:8081)
```

### 배포 명령어

```bash
# 서버에서
cd /srv/fsis2026/deploy
docker compose pull
docker compose up -d

# ghdb superuser 생성 (최초 1회)
docker exec -it ghdb python manage.py createsuperuser
```

## 방화벽 트러블슈팅

ghdb 포트 8001 외부 접근 불가 문제 발생. 원인: 방화벽 다중 레이어.

### 체크해야 할 방화벽 3단계

| 레이어 | 확인 명령 | 설명 |
|--------|-----------|------|
| **1. GCP 방화벽** | `gcloud compute firewall-rules list` | VPC 네트워크 수준. GCP 콘솔에서 인바운드 규칙 추가 |
| **2. ufw** | `sudo ufw status` | OS 레벨 방화벽. `sudo ufw allow 8001/tcp` |
| **3. iptables** | `sudo iptables -L -n` | 저수준 패킷 필터. Docker가 자체 체인 추가하기도 함 |

### 이번 문제의 원인

**ufw에서 8001 포트가 차단되어 있었음.**

GCP 방화벽은 열려 있었으나 ufw에서 막고 있어서 외부 접근 불가.

### 해결

```bash
sudo ufw allow 8001/tcp
sudo ufw reload
```

### 교훈

- 포트 접근 문제 시 **3단계 모두** 확인할 것 (GCP → ufw → iptables)
- GCP 방화벽만 열어서는 안 됨. OS 레벨 방화벽도 별도 관리 필요
- Docker가 iptables를 직접 조작하므로 ufw와 충돌할 수 있음에 유의

## Docker/Nginx 설정 주의사항

### `.env`에 `$` 문자 사용 금지

docker compose가 `$` 이후를 변수로 치환함. `SECRET_KEY`에 `$` 포함 시 값이 깨짐.

```bash
# 안전한 SECRET_KEY 생성 (특수문자 없음)
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

### Docker 볼륨 마운트 시 파일이 미리 존재해야 함

존재하지 않는 파일을 볼륨 마운트하면 Docker가 **디렉토리로 생성**해버림.

```bash
# 잘못된 경우: /srv/ghdb/db_ghdb.sqlite3 가 디렉토리가 됨
# 올바른 순서:
sudo touch /srv/ghdb/db_ghdb.sqlite3   # 먼저 빈 파일 생성
docker compose up -d                    # 그 다음 컨테이너 시작
```
