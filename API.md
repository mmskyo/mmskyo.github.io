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
main.py 생성
```python
from fastapi import FastAP
```