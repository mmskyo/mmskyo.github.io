---
title: apidesc
---
![[Pasted image 20260315200406.png]]

![[api_spec_v1.html]]

# 런타임 & 프레임워크
python 3.11.x
FastAPI 0.111
Pydantic v2
python-jose 3.3
LightGBM 4.x
SHAP 0.45
API v3
tldextract 5.x
POSTGRESQL 16.x
REDIS 7.2.x

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
안드로이드 - 클라이언트
CameraX + ML Kit
