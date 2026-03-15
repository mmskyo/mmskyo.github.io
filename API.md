---
title: api
---
1. 라이브러리 설치
fastapi 실행시,
```bash
pip install fastapi uvicorn
```

2. 데이터 규칙 정하기
프론트엔드와 통신할때는 "어떤 데이터를 보낼지" 규칙을 정해야함
FASTAPI에서는 `pydantic`이라는 도구를 써서 이 규칙(Schema)를 아주 쉽게 만들어줌
```python
from pydantic import BaseModel

# 1. 프론트엔드가 백엔드에 보낼 데이터 규칙 - Request
class ScanRequest(BaseModel):
	url: str       # 검사할 url
	device_id: str # 사용자 기기 id
	
# 2. 백엔드가 프론트엔드에 돌려줄 데이터 규칙 - Response
class ScanResponse(BaseModel):
	url: str # 검사한 url
	is_malicious: bool # 악성 여부(True: 위험, False: 안전)
	score: float # ai가 판단한 위험도 점수(0.0 ~ 1.0)
```

3. 서버 코드 작성 - API 엔드 포인트
main.py 생성하고,
사용자가 접속할 수 있는 '주소(EndPoint)'를 만들어준다.
내 아키텍처에 따라서 Redis 확인 -> ai 추론 -> db 저장 흐름
```python
from fastapi import FastAPI

# FastAPI 앱 생성 (서버의 심장)
app = FastAPI()

# POST 방식으로 '/v1/scan' 이라는 주소를 뚫어줌
@app.post("/v1/scan?, response_model=ScanResponse)
async def scan_url_api(request: ScanRequest):

	#----------
	# [다이어그램 3단계: 비즈니스 로직 & 데이터 처리]
	# 실제 현업이라면 여기에 아래와 같은 로직이 들어감
	#----------
	
	# 1. Redis 캐시 확인 (이전에 똑같은 url을 검사한 적 있는지?)
	# cached_result = check_redis(request.url)
	# if cached_result: return cached_result
	
	# 2. ai 모델 로드 및 추론(s3에서 가져온 lightgbm 사용)
	# ai_score = my_ml_model.predict(request.url)
	
	# 3. RDS 데이터베이스에 스캔 기록 저장
	# save_to_db(request.url, request.device_id, ai_score)
	
	#---------
	
	# 지금은 연습이니까 'bad'가 들어가면 무조건 악성이라 치는 가짜 로직
	is_bad = "bad" in request.url.lower()
	risk_score = 0.99 if is_bad else 0.05
	
	# 프론트엔드에 결과(Response)를 돌려줌
	return ScanResponse(
		url=request.url,
		is_malicious=is_bad,
		score=risk_s)
```