---
title: roadmap
---
---

|주차            |      작업|
|---|---|
|**Week 1-2**   |     API 스펙 전체 확정 (신고 포함) + DB 스키마 확정|
|**Week 3-4**    |    FastAPI 기본 구조 + DB/Redis 연동 + 더미 ML 통합 테스트|
|**Week 5-6**     |   ML 실제 연동 + Whois + Redis 캐싱|
|**Week 7**          |  **신고 API 구현 + S3 피드백 파이프라인 연동**|
|**Week 8-9**      | Android 프론트 — QR 스캔 + 결과 화면 + **신고 UI**|
|**Week 10**      |   Android — 히스토리 / 통계 / 화이트리스트|
|**Week 11**        | 클라우드 배포 + 통합 테스트|
|**Week 12**       |  성능 최적화 + 버그 픽스 + 릴리즈|

# check list
- [ ]  **QR 스캔 화면** (CameraX)
- [ ]  **결과 화면** 안전/위험 분기 + 브라우저 이동 / 다시 스캔
- [ ]  **화이트리스트** CRUD
- [ ]  **히스토리 + 통계** 화면
- [ ]  **신고하기** UI + API 연동 _(신규)_
- [ ]  **FastAPI** `/scan`, `/history`, `/whitelist`, `/report` 4개 라우터
- [ ]  **DB 스키마** `scan_logs`, `whitelist`, `reports`, Materialized View
- [ ]  **Redis 캐싱** URL 캐시 + 중복신고 방지 _(신규)_
- [ ]  **S3 피드백 연동** FALSE_POSITIVE → ML 재학습 파이프라인 _(신규)_
- [ ]  **Room DB** 오프라인 로컬 캐싱


## Backend - Cloud 협의사항:

볼드체 표시 맞는지 확인해주세요!

- [ ] 데이터베이스 호스팅 (**RDS**, CloudSQL, etc)
- [ ] Redis 호스팅 (**ElastiCache**, Memorystore, etc)
- [ ] AI/ML 모델 저장소 (**S3**, GCS, etc)
- [ ] 서버 배포 방식 (**Lambda**, EC2, Cloud Run, AppEngine)
- [ ] 환경변수/시크릿 관리 (**Secrets Manager**, Parameter Store)
- [ ] 로깅 및 모니터링 시스템 (**CloudWatch**, Stackdriver 등), CI/CD 파이프라인 구성, 비용 예측, 자동 스케일링 정책 정의
- [ ] S3 피드백 버킷: `qushing-feedback-data` 버킷 생성 요청 + Lambda IAM 쓰기 권한 부여
- [ ] CloudWatch 알람: 단기간 신고 폭증 시 알람 설정 (DDoS성 악용 방지)
- [ ] 신고 어뷰징 방지: Lambda 레벨 Rate Limiting 또는 API Gateway Throttle 설정

## ML - Backend 협의사항:

- [ ] 모델 저장 형식 (.pkl, .joblib, ONNX 등), 버전 관리 체계, 입출력 데이터 포맷 정의
- [ ] 응답 시간 등 성능 기준, 모델 업데이트 절차, 호스팅 위치 결정 필요
- [ ] 특성 엔지니어링 파이프라인, 데이터 전처리 방식, risk_score 계산을 위한 임계값 설정
- [ ] **피드백 루프**`FALSE_POSITIVE` 신고 데이터를 S3 특정 경로(`s3://bucket/feedback/`)에 쌓고 주기적으로 재학습에 활용
- [ ] **S3 피드백 포맷:** `{"url": "...", "label": "safe", "reported_at": "..."}` JSONL 형식 합의
- [ ] **재학습 트리거:** 피드백 누적 N건 이상 시 CloudWatch 알림 → 재학습 파이프라인 자동 실행 여부
- [ ] **블랙리스트 동기화:** 신고 N회 이상 URL을 ML 학습 데이터에도 `malicious=1`로 추가