---
title: apidesc
---
![[Pasted image 20260315200406.png]]

![[api_spec_v1.html]]

# 점수 산정 로직
vt 점수 환산
```python
vt_raw = (detected / total) * 100
ml_raw = ml_prob * 100

if vt_detected >= 1:
	                vt_weight, ml_weight = 0.8, 0.2
else:               vt_weight, ml_weight = 0.2, 0.8

final score = (vt_raw * vt_weight) + (ml_raw * ml_weight)
```

# 런타임 & 프레임워크
python 3.11.x
FastAPI 0.111
Pydantic v2 # Score breakdown, XAIFeature 모델 정의
python-jose 3.3 # JWT HS256 / bcrypt passlib 1.7
LightGBM 4.x # ml 모델, joblib 직렬화 -> s3 저장
SHAP 0.45 # xai : feature contribution 계산 -> xai_features JSON 저장
API v3 # 바토: httpx async 호출, detected/total -> vt_raw_score
tldextract 5.x # feature 추출: 도메인 파싱 urllib.parse + regex
POSTGRESQL 16.x # sqlalchemy 2.0 async + asyncpg, Alembic 1.13 migration
REDIS 7.2.x # ElastCache - scan:{url_hash} TTL 6시간임
Room / SQLite # 로컬디비


# S3 버킷 구조
```
phishing-detector-bucket/ 
│ 
├── models/ 
│ 
├── v1/ 
│ 
│ 
├── model.pkl # LightGBM joblib 직렬화 
│ 
│ 
├── scaler.pkl # StandardScaler 
│ 
│ 
├── feature_names.json # 피처 순서 (추론 시 필수) 
│ 
│ 
└── metadata.json # { accuracy, f1, trained_at, sample_count } 
│ 
├── v2/ # 재학습마다 버전 증가 
│ 
└── latest/ # 항상 최신 버전 복사본 (Lambda가 여기서 로드) 
│ 
├── model.pkl 
│ 
├── scaler.pkl 
│ 
└── feature_names.json 
│ 
├── datasets/ 
│ 
├── base/ # 초기 학습 원본 (불변) 
│ 
│ 
└── phishing_urls_v1.csv 
│ 
└── approved/ # VT 검증 통과한 피드백 데이터 
│ 
├── feedback_2025Q1.csv 
│ 
└── feedback_2025Q2.csv 
│ 
└── training-logs/ 
└── run_20250310_v2.json # { f1_before, f1_after, promoted: true }
```

# 전체 백엔드 아키텍처
![[Pasted image 20260315203355.png]]

