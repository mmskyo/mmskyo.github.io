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
		score=risk_score
	)
```

4. 서버 실행
```bash
uvicorn main:app --reload
```


질문 -
그럼 url을 스캔하면 제일 먼저 redis cache를 확인하는거야? 로컬은? 
그니까 스캔했는데 화이트리스트에 있다하면 바로 답해주고 미지의 url(히스토리에 없는거)이면 클라우드 진입 후 악성공격을 막는 관문을 거쳐서 lambda모델로 ai가 추론해주고 하는거야? 근데 1. 캐시히트 확인은 대체 뭘하는 작업이야? redis는 로컬 디비처럼 이미 있는걸 빨리 확인할 수 있도록 하는건데 왜 람다를 거쳐서 가? 왜 로그 적재 및 신고내역이 람다를 거쳐서 가는지도 모르겠어. rds가 애초에 따로 있는 이유를 몰라. 거기서 신고데이터만 셀렉해서 람다 트레이닝에서 재학습하는건 맞지?
서버콜드스타트가 어떨때를 말하는지 모르겠어. 재학습된 새 모델이 s3에 저장되어서 콜드 스타트때 람다 모델을 새걸로 업뎃시켜준다는건 이해했어.

답변 정리
맞음!
사용자1이 스캔할 때 로컬에서 처음 체크하고, 화이트리스트면 응답
아니면 레디스, 즉 로컬뿐만 아니라 전 사용자가 공유하는 데이터베이스(휘발성)에 가서도 있는지 체크, 없으면 람다(ai모델)로 추론하는 것
근데 왜 db를 갈때마다 람다를 거치느냐,
그것은 보안과 권한 때문, 애초에 람다는 백엔드로직을 같이 서빙함
직접 db에 접근하면 악성 공격에 무차별해지기에 람다라는 창구를 거침
그렇다면 왜 스캔로그와 신고내역은 따로 rds(postgresql)에 저장하느냐 레디스가 있는데
그것은 둘이 다른 성질의 데이터베이스이기때문
레디스는 고속응답이 가능하지만 메모리가 다차면 기존 데이터를 지워버리는 휘발성 db, 하지만 rds는 영구적으로 저장이되는 db이기에 그동안의 신고내역을 저장하는것. 
콜드스타트는 서버가 잠을 자다 처음으로 깼을 때 준비하는것 시간이 걸림
웜스타트는 한번 깨어난 람다는 바로 자지않고 5-15분 켜져서 대기하기에 초고속 답변 가능
그렇기에 콜드스타트시 새로운 모델을 로딩하는것

또 질문-
근데 그럼 redis의 데이터는 대체 어디에서 난거야? 그리고 어떤 데이터가 들어가? lambda에서 