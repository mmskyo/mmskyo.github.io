---
title: api
nav_order: 5
parent: Projects
---
---
# API 엔드 포인트란
앱이 서버한테 "이거 해줘!"라고 부탁하는 문door
/api/v1/scan 앱이 큐알코드나 바코드를 스캔했을 때 서버한테 보내는 문의 이름
```
안드로이드 앱  →  [ /api/v1/scan 문 ]  →  FastAPI 서버
     📱                🚪                    🖥️
"QR코드 스캔해줘!"         ↓           "스캔 결과 알려줄게!"
                        처리

```
# 실제 동작 과정
```
1️⃣ 안드로이드 앱에서 카메라로 QR코드 찍음
2️⃣ 찍은 사진을 base64로 변환 (문자열)
3️⃣ POST 요청으로 서버에 보냄
   POST /api/v1/scan
   Body: {"image_base64": "iVBORw0KG...", "user_id": "user123"}
4️⃣ 서버(FastAPI)가 QR코드를 해석
5️⃣ 결과 반환
   {"success": true, "data": {"url": "https://google.com"}}
```
# FastAPI
```
from fastapi import FastAPI
app = FastAPI()  # 서버라는 문을 만듦

@app.post("/api/v1/scan")  # "/api/v1/scan 문"을 만듦
async def scan(request: ScanRequest):  # 누가 문을 두드리면 이 함수가 실행
    # request = 앱이 보낸 데이터 (사진+유저ID)
    
    # 실제 QR 해석 로직 (나중에 구현)
    result = decode_qr_code(request.image_base64)
    
    return {"success": True, "data": result}  # 앱한테 답장

```

## 핵심
```
@app.post("/api/v1/scan")  # 어떤 문인지
async def scan(request):    # 문 두드리면 실행될 함수
    return {"결과"}         # 문 열고 답장
```

# 요청 응답 전체 흐름
```안드로이드 앱이 보냄 ↓
POST /api/v1/scan HTTP/1.1
Host: your-server.com
Content-Type: application/json

{
  "image_base64": "/9j/4AAQSkZJRgABAQE...",
  "user_id": "kakao12345",
  "scan_type": "qr"
}

FastAPI가 답장 ↑
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "data": {
    "type": "QR",
    "url": "https://github.com",
    "confidence": 0.98
  }
}

```
# API
REST API = 집 문의 규칙 (어떻게 문을 만들어야 하는지) 규칙/표준 ex. GET=읽기, POST=만들기
FastAPI = 문 제작 회사 (실제로 문을 만들어주는 도구) 도구/프레임워크 ex. @app.post("/scan")

restapi만 있으면 원시상태
```python
# 수동으로 http 요청 처리 (flask 기본형)
@app.route("/scan", methods=["POST"])
def scan():
	raw_data = request.get_data() # 바이트로 받아서
	json_data = json.loads(raw_data) # 수동 파싱
# 수동 검증, 수동 에러처리 ...
return json.dumps({"result": "ok"}) # 수동 변환
```
fastapi 도구가 규칙 자동 적용
```python
@app.post("/scan") #REST 규칙 자동 적용
async def scan(request: ScanRequest): # 자동 검증
	return request # 자동 JSON 변환
```

왜 restapi?
1. "안드로이드 앱이 서버와 통신" = REST API 규칙 필요
2. FastAPI는 그 규칙을 "쉽게 구현해주는 도구"
3. FastAPI 없이도 REST API 만들 수 있음 (Flask, Django DRF)

요청 예시
GET    /api/users/123    → 사용자 정보 가져오기
POST   /api/scan         → QR 스캔 요청
PUT    /api/users/123    → 사용자 정보 수정
DELETE /api/scan/456     → 스캔 기록 삭제

fastapi는 restapi를 자동으로 만들어줌
```python
from fastapi import FastAPI
app = FastAPI()  # REST 규칙 적용된 서버 생성

@app.get("/users/{id}")      # REST GET 자동
@app.post("/scan")           # REST POST 자동
@app.put("/users/{id}")      # REST PUT 자동
@app.delete("/scan/{id}")    # REST DELETE 자동

```
FastAPI가 해주는 일:
	•	HTTP 메서드 자동 처리
	•	JSON 자동 변환
	•	데이터 검증 자동
	•	 /docs  자동 문서화
	•	에러 처리 자동

API 스펙

| 엔드포인트           | 메서드    | 역할          | 앱 화면   |
| --------------- | ------ | ----------- | ------ |
| /api/v1/scan    | `POST` | QR 스캔 + 검사  | 메인기능   |
| /api/v1/history | `GET`  | 검사 이력 조회    | 히스토리 탭 |
| /api/v1/stats   | `GET`  | 통계(이번주 안전률) | 대시보드   |

# 실제 구현 코드
```python
from fastapi import FastAPI, HTTPException, Depends
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import Optional, List
import hashlib
import time
from enum import Enum

app = FastAPI(title="QR Safe Scanner API", version="1.0.0")

# CORS - 안드로이드 앱 허용
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 나중에 앱 도메인으로 변경
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 요청/응답 데이터 형식
class ScanType(str, Enum):
    QR = "qr"
    BARCODE = "barcode"

class ScanRequest(BaseModel):
    image_base64: str        # 앱에서 보낸 QR 이미지
    user_id: str             # 사용자 식별자
    scan_type: ScanType = ScanType.QR

class ScanResult(BaseModel):
    safe: bool               # 안전여부
    url: Optional[str] = None  # QR에서 추출된 URL
    risk_score: float        # 위험도 0.0~1.0
    risk_reason: Optional[str] = None  # "피싱", "악성코드"
    scan_id: str             # 추적용 ID

class HistoryResponse(BaseModel):
    scans: List[ScanResult]

# 가짜 악성 URL 데이터베이스 (실제론 외부 API 연동)
DANGEROUS_SITES = {
    "http://malware.com": 0.95,
    "http://phishing.kr": 0.92,
    "http://fakebank.com": 0.88
}

# 1️⃣ 메인 스캔 API (당신 핵심 역할)
@app.post("/api/v1/scan", response_model=ScanResult)
async def scan_qr(request: ScanRequest):
    """
    QR코드 스캔 → URL 추출 → 악성여부 검사 → 결과 반환
    """
    # 1. QR에서 URL 추출 (실제론 pyzbar/opencv 사용)
    extracted_url = extract_url_from_qr(request.image_base64)
    
    # 2. 악성 사이트 검사
    risk_score = check_malware(extracted_url)
    
    # 3. 결과 판단
    is_safe = risk_score < 0.5
    risk_reason = "피싱 사이트" if risk_score > 0.8 else None
    
    # 4. 고유 scan_id 생성
    scan_id = hashlib.md5(f"{request.user_id}{time.time()}".encode()).hexdigest()[:8]
    
    return ScanResult(
        safe=is_safe,
        url=extracted_url,
        risk_score=risk_score,
        risk_reason=risk_reason,
        scan_id=scan_id
    )

# 2️⃣ 검사 이력 조회
@app.get("/api/v1/history", response_model=HistoryResponse)
async def get_history(user_id: str):
    """사용자의 최근 10개 검사 이력"""
    # 실제론 Redis/DB에서 조회
    return HistoryResponse(scans=[
        ScanResult(
            safe=True, 
            url="https://naver.com", 
            risk_score=0.1, 
            scan_id="abc12345"
        )
    ])

# 3️⃣ 통계
@app.get("/api/v1/stats")
async def get_stats(user_id: str):
    """이번주 안전률 등 통계"""
    return {
        "safe_rate": 0.92,
        "total_scans": 47,
        "risky_scans": 4
    }

# 헬퍼 함수들 (실제 구현 시 외부 라이브러리 사용)
def extract_url_from_qr(image_base64: str) -> str:
    """QR 이미지에서 URL 추출 (가짜)"""
    # 실제론: pyzbar, opencv-python 사용
    return "https://naver.com"  # 테스트용

def check_malware(url: str) -> float:
    """악성 여부 점수 반환 (가짜)"""
    # 실제론: Google Safe Browsing API, VirusTotal API 연동
    if any(danger in url.lower() for danger in DANGEROUS_SITES):
        return DANGEROUS_SITES.get(url, 0.1)
    return 0.1  # 기본 안전

# 서버 상태 확인
@app.get("/")
async def root():
    return {"message": "QR Safe Scanner API 동작 중", "docs": "/docs"}

```

# 안드로이드에서 호출하는 법
```kotlin
// Retrofit 인터페이스
interface ScanApi {
    @POST("api/v1/scan")
    suspend fun scanQr(@Body request: ScanRequest): ScanResult
}

// 실제 호출
val request = ScanRequest(
    imageBase64 = base64Image,
    userId = "user123",
    scanType = "qr"
)
val result = api.scanQr(request)

// 앱 화면 처리
when {
    result.safe -> showSafeScreen(result.url)  // [안전][이동]
    else -> showDangerScreen(result.url, result.riskScore)  // [홈][무시하고 이동]
}

```

# 서버에서 ml과 연동하여 실행하는 법
main.py
```python
from fastapi import FastAPI, HTTPException

from pydantic import BaseModel

import joblib # scikit-learn 모델용

import torch # PyTorch 모델용

import numpy as np

from typing import Optional

  

app= FastAPI(title="QR Safe Scanner with ML")

  

# 1단계 : ML 모델 로드 (서버 시작 시 1회)

try:

# 모델 1 scikit-learn 모델 .pkl

ml_model= joblib.load("malware_classifier.pkl")

print("모델 로드 완료 scikit learn")

  

# 모델 2 pytorch .pt

  

except Exception as e:

print(f"모델 로드 실패: {e}")

ml_model= None

  
  

# 스캔 요청

class ScanRequest(BaseModel):

image_base64: str

user_id: str

extracted_url: Optional[str]= None # 다른 분이 미리 디코딩

  

# ML 모델에 직접 보낼 입력(협의)

class MLInferenceRequest(BaseModel):

url_features: list[float] # [도메인 길이, ip 포함 여부, https 여부 등]

# 팀원1이 알려준 입력 형식에 맞추기

  

class MLResponse(BaseModel):

safe: bool

risk_score: float

confidence: float

  

# 2단계 : QR 스캔->ML추론 파이프라인

@app.post("/api/v1/scan", response_model=MLResponse)

async def scan_qr_ml(request: ScanRequest):

if ml_model is None:

raise HTTPException*500, "모델 로드 실패")

# 1. QR 디코딩

url= request.extracted_url or "https://naver.com"

  

# 2. URL -> ML 입력 피처 변환

url_features= url_to_features(url)

  

# 3. ML 모델 추론

risk_score= ml_predict(url_features)

  

# 4. 결과 판단

is_safe= risk_score < 0.5

confidence= 0.95 # 모델의 confidence score

  

return MLResponse(

safe= is_safe,

risk_score= risk_score,

confidence= confidence

)

  

# 3단계: ML 예측 함수들

def ml_predict(features: list[float]) -> float:

"""ML 모델 추론 실행"""

try:

# scikit learn

feature_array= np.array([features]).reshape(1, -1)

prediction= ml_model.predict_proba(feature_array)[0][1] # 클래스 1(위험) 확률

return float(prediction)

  

# pytorch

  

except Exception as e:

print(f"ML 추론 오류: {e}")

return 0.1 # 기본 안전

  

# 테스트용 ML 엔드포인트

@app.post("/api/v1/ml/predict", response_model= MLResponse)

async def test_ml_model(request: MLInferenceRequest):

"""ML 모델 직접 테스트"""

risk_score= ml_predict(request.url.features)

return MLResponse(safe= risk_score < 0.5, risk_score= risk_score, confidence= 0.95)

  

@app.get("/health")

async def health_check():

return {

"status": "healthy"

"ml_model_loaded": ml_model is not None
}
```

- requirements.txt 추가
```python
fastapi==0.115.0
uvicorn[standard]==0.32.0
scikit-learn==1.5.1
torch==2.4.0
numpy==1.26.4
joblib==1.4.2
python-multipart==0.0.9
```

# 실행 및 배포 순서
```bash
#1.의존성 설치
pip install fastapi uvicorn scikit-learn numpy torch hoblib

#2. ML 모델 파일 서버 폴더에 복사
malware_classifier.pkl # 파일을 루트에 넣기

#2. 실행
pip install "fastapi[standard]" # fastapi dev 명령어 쓰려면 설치
fastapi dev main.py

#4. 테스트
curl -X POST "http://localhost:8000/api/v1/scan" \
	-H "Content-Type: application/json" \
	-d '{image_base64": "test", "user_id": "user1", "extracted_url": "https://naver.com"}'
```

# 디버깅 체크 리스트
| 단계    | 확인사항                                       | 상태  |
| ----- | ------------------------------------------ | --- |
| 모델 로드 | `GET /health` -> `"ml_model_loaded": true` |     |
| 피처 변환 | `url_to_features()`출력 확인                   |     |
| 추론    | `POST /api/v1/ml/predict`성공                |     |
| 통합    | `POST /api/v1/scan`전체 동작                   |     |

# 실무 팁
1. 모델 로드는 서버 시작 시 1회(메모리 상주)
2. 입력 피처 순서 1개만 틀려도 100% 실패
3. /healtch 엔드포인트 필수 (배포 후 모니터링)
4. GPU 필요시 docker + nvidia-runtime