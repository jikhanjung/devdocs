# 061. 추출 파이프라인 파일 구조 개편 및 스크립트 정리

**날짜**: 2026-04-06  
**버전**: 0.3.24 → 0.3.25

---

## 1. 연도별 현황 페이지 통합 파이프라인 카운트 수정 (0.3.24)

**원인**: `/kprdb/pdf_status_by_year/`에서 통합 파이프라인(Claude 처리 완료) 판단을 `'figures' in edata`로 하고 있었음. 그런데 regex-only 결과도 `"figures": []`를 포함하므로 → regex만 처리된 논문도 통합 완료로 잘못 카운트됨.

**수정**: `'claude_model' in edata`로 변경 — Claude 처리 완료 시에만 기록되는 필드.

---

## 2. 추출 결과 파일 구조 개편 (0.3.25)

### 기존 구조의 문제

- `.pdf.extract.json` 하나에 regex + Claude 결과가 덮어쓰기로 합쳐짐
- 파일 존재 여부만으로 처리 단계 구분 불가

### 새 파일 구조

| 파일 | 생성 시점 | 내용 |
|---|---|---|
| `.pdf` | PDF 업로드 | 원본 PDF |
| `.pdf.json` | OCR 완료 | RunPod OCR 결과 |
| `.pdf.extract.json` | regex 추출 완료 | 좌표/taxa/표본 (regex) |
| `.pdf.extract.claude.json` | Claude 보강 완료 | 위 내용 + figures (Claude) |

### 마이그레이션

`migrate_claude_extract.py` 실행 — `.extract.json` 중 `claude_model` 필드 있는 420개를 `.extract.claude.json`으로 복사.

```bash
python scripts/migrate_claude_extract.py /srv/fsis2026/uploads/references
# copied: 420, skipped (no claude): 614
```

---

## 3. `extract_from_ocr.py` 수정

- `process_json()` — Claude 보강 블록 제거, `(regex_result, pages)` 튜플 반환으로 변경
- `_make_summary()` 헬퍼 함수 분리
- `--claude`, `--claude-model` 옵션 제거 → regex-only 전용 스크립트로 단순화

---

## 4. `claude_augment.py` 신규 작성

Claude 보강 전용 스크립트. `.pdf.extract.json` + `.pdf.json` 읽어 Claude 처리 후 `.pdf.extract.claude.json` 저장.

```bash
python scripts/claude_augment.py /srv/fsis2026/uploads/references
python scripts/claude_augment.py /srv/fsis2026/uploads/references --model sonnet
python scripts/claude_augment.py /srv/fsis2026/uploads/references --limit 1
```

- `.extract.claude.json` 이미 있으면 스킵
- rate limit 시 중단, 이미 처리된 건 보존
- lockfile(`/tmp/claude_augment.lock`) — 동시 실행 방지
- `--limit` — 처리 수 기준 (스킵 파일 제외)

---

## 5. views.py / views_task.py 수정

`_best_extract_path(pdf_path)` 헬퍼 추가:

```python
def _best_extract_path(pdf_path: str) -> str:
    claude = pdf_path + '.extract.claude.json'
    return claude if os.path.exists(claude) else pdf_path + '.extract.json'
```

6곳 일괄 적용 (views.py 5곳, views_task.py 2곳). Claude 처리 완료 논문은 자동으로 더 풍부한 데이터 표시.

`pdf_status_by_year` — `extract`는 `.extract.json` 존재 여부, `unified`는 `.extract.claude.json` 존재 여부로 명확히 분리.

---

## 6. 구 `.claude-extract.json` 코드 제거

deprecated된 `extract_claude.py`가 만들던 구 포맷 파일(서버에 2개만 존재).
- `views.py` — `claude_extract_data` 블록 제거
- `pdf_detail.html` — `claudeFigs` 폴백 로직, "Claude 추출 비교" 패널 제거

---

## 7. 미사용 스크립트 정리

삭제:
- `scripts/split_subfigures.py` — 미사용 (CLAUDE.md에 미사용 표시)
- `scripts/fix_coord_precision.py` — 일회성 스크립트, 완료

`scripts/deprecated/`로 이동:
- `extract_claude.py` — deprecated (기능이 `extract_from_ocr.py --claude`에 통합됐었으나 이제 `claude_augment.py`로 분리)
- `batch_runpod_seq.py` — 순차처리 버전, 미사용
- `migrate_claude_extract.py` — 일회성 마이그레이션, 완료

`/srv/fsis2026/scripts/extract_claude.py` 삭제.

---

## 8. `claude_augment.py` cron 등록

```
*/10 * * * * /srv/fsis2026/venv/bin/python /srv/fsis2026/scripts/claude_augment.py /srv/fsis2026/uploads/references --limit 1 >> /srv/fsis2026/scripts/claude_augment.log 2>&1
```

10분마다 1개씩, 오래된 논문부터(경로 오름차순 = 연도 오름차순) 처리. 621개 미처리 논문 대상.

---

## 9. `build.sh` 스크립트 복사 자동화

빌드 시 `scripts/*.py`를 `/srv/fsis2026/scripts/`에 자동 복사하는 단계 추가 (3/5단계).

```bash
cp "$PROJECT_DIR"/scripts/*.py /srv/fsis2026/scripts/
```

---

## 변경 파일

- `kprdb/views.py` — `_best_extract_path()` 추가, 통합 파이프라인 카운트 수정, `claude_extract_data` 제거
- `kprdb/views_task.py` — `_best_extract_path()` 적용
- `kprdb/templates/kprdb/pdf_detail.html` — 구 Claude 비교 패널 제거
- `scripts/extract_from_ocr.py` — regex-only 전용으로 단순화
- `scripts/claude_augment.py` — 신규
- `scripts/migrate_claude_extract.py` → `scripts/deprecated/`
- `scripts/split_subfigures.py` — 삭제
- `scripts/fix_coord_precision.py` — 삭제
- `deploy/build.sh` — scripts 복사 단계 추가

## 커밋

- `3b0f148` — Bump version to 0.3.24 (통합 파이프라인 카운트 수정)
- `54828a4` — Bump version to 0.3.25 (추출 파이프라인 개편 + 스크립트 정리)
