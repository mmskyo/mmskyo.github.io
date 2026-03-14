---
title: database
nav_order: 6
parent: Projects
---
---
# DB 스키마
## scans 테이블 (메인)
| id  | user_id | url | safe | risk_score | scan_time | device_info |
| --- | ------- | --- | ---- | ---------- | --------- | ----------- |
|     |         |     |      |            |           |             |
## url_threats 테이블 (악성 url 집합)
| id  | url_hash | threat_level | first_seen | last_seen | count |
| --- | -------- | ------------ | ---------- | --------- | ----- |
|     |          |              |            |           |       |

## user_stats 테이블 (사용자 통계)
| user_id | total_scans | safe_scans | risky_scans | last_active |
| ------- | ----------- | ---------- | ----------- | ----------- |
|         |             |            |             |             |

# DB 연동 코드 (프로젝트 구조)
fastapi-server/
├── app/
 │           ├── __init__.py
 │            ├── main.py
 │   ├── database.py
 │   ├── models.py
 │   └── schemas.py
├── requirements.txt
└── .env