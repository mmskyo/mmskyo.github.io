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
 │           ├── main.py
 │           ├── database.py
 │           ├── models.py
 │           └── schemas.py
├── requirements.txt
└── .env

1. database.py - 데이터베이스 연결 설정

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

# .env에서 DB URL 읽기
DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./scans.db")

engine = create_engine(
    DATABASE_URL, 
    connect_args={"check_same_thread": False}  # SQLite용
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# DB 세션 의존성 주입
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

2. models.py - DB 테이블 정의
```python
from sqlalchemy import Column, Integer, String, Float, DateTime, Boolean
from sqlalchemy.sql import func
from .database import Base

class ScanLog(Base):
    __tablename__ = "scans"
    
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String, index=True)
    url = Column(String)
    safe = Column(Boolean)
    risk_score = Column(Float)
    scan_time = Column(DateTime(timezone=True), server_default=func.now())
    device_info = Column(String, nullable=True)

class ThreatURL(Base):
    __tablename__ = "url_threats"
    
    id = Column(Integer, primary_key=True, index=True)
    url_hash = Column(String, unique=True, index=True)
    threat_level = Column(Float)
    first_seen = Column(DateTime(timezone=True), server_default=func.now())
    last_seen = Column(DateTime(timezone=True), server_default=func.now())
    detection_count = Column(Integer, default=1)
```

3. schemas.py - API 요청/응답 형식
```python
from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

class ScanCreate(BaseModel):
    image_base64: str
    user_id: str
    extracted_url: Optional[str] = None

class ScanResult(BaseModel):
    id: int
    safe: bool
    url: str
    risk_score: float
    scan_time: datetime
    
    class Config:
        from_attributes = True  # ORM 객체 → Pydantic 변환

class HistoryResponse(BaseModel):
    scans: List[ScanResult]
    total: int
```

4. main.py - DB 연동된 완전한 API
```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
import hashlib
from . import database, models, schemas
from .database import engine

app = FastAPI(title="QR Safe Scanner API v1")

# 테이블 자동 생성
models.Base.metadata.create_all(bind=engine)

@app.post("/api/v1/scan", response_model=schemas.ScanResult)
async def scan_qr(request: schemas.ScanCreate, db: Session = Depends(database.get_db)):
    """QR 스캔 → DB 저장 → 결과 반환"""
    
    # 1. URL 추출 (팀원2 목업)
    url = request.extracted_url or "https://naver.com"
    
    # 2. ML 예측 (목업 - 나중에 실제 모델로 교체)
    risk_score = 0.1  # ml_predict(url)
    is_safe = risk_score < 0.5
    
    # 3. DB에 스캔 로그 저장
    scan_log = models.ScanLog(
        user_id=request.user_id,
        url=url,
        safe=is_safe,
        risk_score=risk_score,
        device_info="Android 14"
    )
    db.add(scan_log)
    db.commit()
    db.refresh(scan_log)
    
    # 4. 악성 URL 집합 업데이트 (옵션)
    url_hash = hashlib.md5(url.encode()).hexdigest()
    threat_url = db.query(models.ThreatURL).filter(models.ThreatURL.url_hash == url_hash).first()
    if not threat_url:
        threat_url = models.ThreatURL(
            url_hash=url_hash,
            threat_level=risk_score
        )
        db.add(threat_url)
    else:
        threat_url.detection_count += 1
        threat_url.last_seen = func.now()
    db.commit()
    
    return scan_log

@app.get("/api/v1/history", response_model=schemas.HistoryResponse)
async def get_history(user_id: str, limit: int = 10, db: Session = Depends(database.get_db)):
    """사용자 스캔 이력 조회"""
    scans = db.query(models.ScanLog)\
        .filter(models.ScanLog.user_id == user_id)\
        .order_by(models.ScanLog.scan_time.desc())\
        .limit(limit).all()
    
    return schemas.HistoryResponse(
        scans=scans,
        total=len(scans)
    )

@app.get("/api/v1/stats/{user_id}")
async def get_stats(user_id: str, db: Session = Depends(database.get_db)):
    """사용자 통계"""
    total = db.query(models.ScanLog).filter(models.ScanLog.user_id == user_id).count()
    safe = db.query(models.ScanLog).filter(
        models.ScanLog.user_id == user_id, 
        models.ScanLog.safe == True
    ).count()
    
    return {
        "user_id": user_id,
        "total_scans": total,
        "safe_rate": safe / total if total > 0 else 0,
        "risky_scans": total - safe
    }
```
## 실행 방법
```bash
# 1. requirements.txt
echo "fastapi uvicorn sqlalchemy sqlmodel python-dotenv" > requirements.txt

# 2. .env 파일 생성 (SQLite 무료)
echo "DATABASE_URL=sqlite:///./scans.db" > .env

# 3. 폴더 구조 만들고 위 코드들 저장

# 4. 실행
fastapi dev main.py
```

브라우저 테스트:
```http://localhost:8000/docs
→ POST /api/v1/scan → {"image_base64":"test", "user_id":"user123"}
→ GET /api/v1/history?user_id=user123
→ GET /api/v1/stats/user123

```

배포용 PostgreSQL (.env 수정)
DATABASE_URL=postgresql://user:pass@your-render-db:5432/scans

DB 확인
```sql
-- SQLite 파일 열기
sqlite3 scans.db

-- 데이터 확인
SELECT * FROM scans ORDER BY scan_time DESC LIMIT 5;
SELECT COUNT(*) FROM url_threats WHERE threat_level > 0.5;

```

