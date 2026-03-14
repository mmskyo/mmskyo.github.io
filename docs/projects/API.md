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
    result.safe -> showSafeScreen(result.url)  // [안전 ✅][이동 🔗]
    else -> showDangerScreen(result.url, result.riskScore)  // [위험 ⚠️][홈↩️][무시➡️]
}

```
