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
fastapi dev app/main.py
```

브라우저 테스트:
```http://localhost:8000/docs
→ POST /api/v1/scan → {"image_base64":"test", "user_id":"user123"}
→ GET /api/v1/history?user_id=user123
→ GET /api/v1/stats/user123

```

# 배포용 PostgreSQL (.env 수정)
DATABASE_URL=postgresql://user:pass@your-render-db:5432/scans

DB 확인
```sql
-- SQLite 파일 열기
sqlite3 scans.db

-- 데이터 확인
SELECT * FROM scans ORDER BY scan_time DESC LIMIT 5;
SELECT COUNT(*) FROM url_threats WHERE threat_level > 0.5;

-- 나가기
.exit
.quit
```

# 스키마
users 테이블
├── id (Primary Key, Auto Increment) 
├── email (Unique + Index) 
├── hashed_password (VARCHAR(255)) 
└── created_at (Timestamp, Default now()) 

scans 테이블 (Foreign Key 관계)
├── id (PK) 
├── user_id (FK → users.id) 
├── url (VARCHAR) 
├── safe (Boolean) 
├── risk_score (Float) 
└── scan_time (Timestamp) 

# ORM 연동 (SQLAlchemy)
1. Session 관리: Depends(get_db) → 자동 생성/종료 ✓
2. 모델 정의: declarative_base() → 테이블 자동 생성 ✓
3. 쿼리: session.query() + filter() → 성능 최적화 ✓
4. 트랜잭션: add → commit → refresh → 완벽 ✓
ex.
```
# 1. 세션 주입 (의존성 주입)
async def scan(..., db: Session = Depends(get_db)):

# 2. 쿼리 (인덱스 활용)
db_user = db.query(User).filter(User.email == email).first()

# 3. 저장 (트랜잭션 자동)
db.add(user)
db.commit()  # 자동 롤백 처리
db.refresh(user)  # 최신 데이터

# 4. 응답 변환 (Pydantic 자동)
return user  # ORM → JSON
```


# 비동기식 생성으로 수정
참고: https://devspoon.tistory.com/308


# 데이터베이스란..
1. RDBMS vs PostgreSQL (카테고리 vs 브랜드)
rdbms의 종류이다..
이게 rds 구역에 설치되는 진짜 데이터베이스 엔진

2. ORM vs SQLAlchemy (기술개념 vs 도구)
백엔드 코드와 db를 연결하는 부분
- orm;object relational mapping : 
		객체object와 관계relational를 연결mapping한다 는 기술적 개념
		파이썬의 Class와 DB의 Table을 똑같이 생기게 만들어서 sql을 몰라도 파이썬 코드로 db를 다룰 수 있게 해주는 마법같은 방법론
- SQLAlchemy: 파이썬에서 orm을 구현할 수 있게 도와주는 가장 유명한 라이브러리
		백엔드구현: 파이썬으로 User.save()라고 쓰면 알케미가 이걸 INSERT INTO users 라는 sql문으로 번역해서 PostgreSQL에게 전달

2. ER vs RDBMS (설계도 vs 완성건물)
어떤 단계에서 쓰느냐!
	- ER: 데이터를 설계할 때 그리는 기획서/설계도
	- RDBMS: 그 설계도를 바탕으로 실제로 데이터를 저장할 수 있게 구축된 완성된 건물

# SQLAlchemy 맛보기
db에 url 스캔 결과를 저장하는 테이블을 만들고, 데이터를 하나 집어넣어라 는 명령
```python
from sqlalchemy import create_all, Column, Integer, String, Float, Boolean
from sqlalchemy 
```

# 무중단 갱신 Hot Reload
서버를 끄지않고 손님들 모르게 모델을 바꿔치기
lambda_train이 APIGW를 호출하면서 무중단 갱신을 요청
lambda_api 일꾼들이 아직도 예전 툴model_v1.pkl을 들고 일을함
이때 백엔드 시스템은 기존 일꾼들을 퇴근시키고 새로운 모델을 든 새 일꾼을 투입

작동 원리 : 백엔드코드에서 s3의 latest 폴더에 있는걸 가져와라 고 짜놨으면 lambda train이 s3파일만 쓱 갈아치움

방법 2가지
1. s3 경로 활용
2. lambda의 버전/별칭(alias) 기능 사용
기존 서비스에 영향 x

환경변수 활용 - MODEL_PATH에 s3 주소 적기
Redis에 최신버전 정보 저장 - 레디스에 curent_model_path 키로 주소 저장 -> 람다 재부팅 필요 없음 but, 새 모델 로드마다 s3에서 다운로드하는 로직 코딩
