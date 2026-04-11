# GHDB (Geological Heritage Database)

2026 지질유산 데이터베이스. 지질유산 유물(화석 및 암석) 관리를 위한 별도 Django 앱.

## 배경

2026년 지질유산 프로젝트를 위해 생성. FSIS의 MuseumSpecimen 모델을 확장하여 화석과 암석 두 가지 표본 유형을 지원한다.

## 주요 모델

| 모델 | 역할 |
|---|---|
| **MuseumSpecimen** | 표본 정보 (fossil/rock 두 유형 지원) |

### 암석 전용 필드 (7개)
- rock_category, rock_name, texture, structure, color, description
- source_system (DB 통합 추적용)

## 기능

- **표본 CRUD**: 화석/암석별 특화 폼, JS 토글로 fossil/rock/trace 섹션 전환
- **대량 임포트**: ZIP 업로드 (Excel + 이미지, prefix 매칭), 3가지 표본 유형 지원
- **사용자 관리**: 프로필, 비밀번호 변경, 관리자 사용자 관리
- **로그인 페이지**: GET 방식 지원
- **샘플 데이터**: 화석 10건 (GPS/사진 포함), 암석 5건

## 배포

- 서버: 34.64.158.160, 포트 8001
- 3중 방화벽 해결: GCP firewall + ufw + iptables
- Docker compose, Nginx reverse proxy
- HTTPS: Let's Encrypt (비표준 포트 수동 SSL 설정)
- 브랜딩: "2026 지질유산 데이터베이스"

## Docker 최적화

- .dockerignore로 550MB map_tiles 제외
- testdata 디렉토리 제거로 1.56GB → 473MB 이미지 크기 축소

## 스모크 테스트

7건의 ghdb 전용 테스트 포함 (전체 61건 중)

## 관련 페이지

- [배포 인프라](deployment.md)
- [프로젝트 개요](fsis2026-overview.md)

---
*Sources: 020, 021, 022, 023, 024, 026, 027, 033, P15*
