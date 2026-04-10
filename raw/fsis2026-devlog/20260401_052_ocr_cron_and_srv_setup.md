# 052. OCR 자동화 cron 설정 및 /srv/fsis2026 구조 정리

**날짜**: 2026-04-01
**버전**: 0.3.2

## 1. 프로덕션 서버 디렉토리 구조

```
/srv/fsis2026/
├── db.sqlite3              # 프로덕션 DB
├── deploy.sh               # Docker 배포 스크립트
├── docker-compose.yml      # Docker 서비스 설정
├── logs/                   # 로그
├── map_tiles/              # 지도 타일 (Nginx 직접 서빙)
├── staticfiles/            # Django collectstatic 결과
├── uploads/                # MEDIA_ROOT
│   └── references/         # 논문 PDF + OCR JSON + 추출 결과
│       └── {year}/
│           ├── {id}.pdf
│           ├── {id}.pdf.json           # OCR 결과 (Chandra2)
│           ├── {id}.pdf.extract.json   # regex 추출 결과
│           └── {id}.pdf.claude-extract.json  # Claude 추출 결과 (선택)
├── venv/                   # Python 가상환경 (서버 전용)
│   └── bin/python
└── scripts/                # 서버 전용 스크립트 (git 외부)
    ├── .env                # RunPod 인증 (RUNPOD_API_KEY, RUNPOD_ENDPOINT_ID)
    ├── .last_ocr_run       # cron 마지막 실행 타임스탬프
    ├── batch_runpod.py     # RunPod OCR 배치 처리 (프로젝트 scripts/ 복사본)
    ├── extract_from_ocr.py # regex 기반 정보 추출 (프로젝트 scripts/ 복사본)
    ├── extract_claude.py   # Claude CLI 기반 추출 (프로젝트 scripts/ 복사본)
    ├── ocr_cron.sh         # cron 실행 스크립트
    └── ocr_cron.log        # cron 실행 로그
```

## 2. /srv/fsis2026/venv 설정

Docker 컨테이너와 분리된 서버 전용 가상환경. OCR 스크립트 실행에 필요한 패키지만 설치.

```bash
python3 -m venv /srv/fsis2026/venv
/srv/fsis2026/venv/bin/pip install PyMuPDF Pillow requests python-dotenv
```

설치된 패키지:
- PyMuPDF 1.27.2 — PDF 페이지 렌더링 (base64 JPEG 변환)
- Pillow 12.1.1 — 이미지 처리
- requests 2.33.1 — RunPod API 호출
- python-dotenv 1.2.2 — .env 파일 로드

## 3. /srv/fsis2026/scripts 설정

프로젝트 `scripts/` 디렉토리에서 복사:

```bash
cp scripts/batch_runpod.py scripts/extract_from_ocr.py scripts/extract_claude.py scripts/.env /srv/fsis2026/scripts/
```

**보안 설계**: RunPod API 키는 `/srv/fsis2026/scripts/.env`에만 존재.
Docker 컨테이너(웹)에는 키가 포함되지 않음. OCR 처리는 호스트에서만 실행.

스크립트 업데이트 시 수동 복사 필요:
```bash
cp scripts/*.py /srv/fsis2026/scripts/
```

## 4. ocr_cron.sh — 자동 OCR 처리

### 동작 방식

1. **증분 처리** (기본): `.last_ocr_run` 타임스탬프 이후 변경된 PDF만 체크
2. **전체 스캔** (`--full`): 모든 PDF를 체크
3. 미처리 PDF 발견 시:
   - 임시 디렉토리에 symlink 생성 (새 파일만)
   - `batch_runpod.py`로 OCR 실행
   - 결과 JSON을 원래 위치로 복사
   - `extract_from_ocr.py`로 추출 실행
4. 타임스탬프 갱신

### 안전장치

- **lockfile** (`/tmp/ocr_cron.lock`): 동시 실행 방지
- PID 체크: 이전 프로세스가 살아있으면 스킵
- 미처리 PDF가 없으면 즉시 종료 (리소스 낭비 방지)

### cron 등록

```bash
# 10분마다 실행
crontab -e
*/10 * * * * /srv/fsis2026/scripts/ocr_cron.sh >> /srv/fsis2026/scripts/ocr_cron.log 2>&1
```

### 수동 실행

```bash
# 증분 (새 파일만)
/srv/fsis2026/scripts/ocr_cron.sh

# 전체 스캔
/srv/fsis2026/scripts/ocr_cron.sh --full

# 로그 확인
tail -f /srv/fsis2026/scripts/ocr_cron.log
```

## 5. Management command — ocr_process

Django management command로도 OCR 실행 가능 (호스트에서):

```bash
DATABASE_PATH=/srv/fsis2026/db.sqlite3 MEDIA_ROOT=/srv/fsis2026/uploads \
  /home/honestjung/venv/fsis2026/bin/python manage.py ocr_process

# 특정 논문만
python manage.py ocr_process --ref-id 2998

# 미리보기
python manage.py ocr_process --dry-run
```

**주의**: Docker 컨테이너 안에서는 RunPod 키가 없으므로 실행 불가.
반드시 호스트에서 실행해야 함.

## 6. PDF 업로드 → OCR 처리 흐름

1. 사용자가 워크스페이스에서 PDF 업로드 → Django `api_upload_pdf` → DB 저장
2. cron (10분 간격)이 새 PDF 감지 → RunPod OCR → `.pdf.json` 생성
3. 이어서 `extract_from_ocr.py` → `.pdf.extract.json` 생성
4. 다음 페이지 로드 시 추출 데이터 표시

## 7. 기타

- `api_upload_pdf`에서 백그라운드 subprocess 호출 코드 제거 (컨테이너에 키 없음)
- PDF 업로드 시 `ref.save()` (history 기록, `save_without_historical_record` 에서 변경)
- 3611.pdf OCR 완료 → 967/967 전체 OCR 완료
- RunPod max workers: 8, active: 0 (서버리스), endpoint: `2vk4pcv7kkn2ax`
