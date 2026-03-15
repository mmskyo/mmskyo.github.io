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