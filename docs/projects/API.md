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
