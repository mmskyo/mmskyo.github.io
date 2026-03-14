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
