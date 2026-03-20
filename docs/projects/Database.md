---
title: database
nav_order: 6
parent: Projects
---
---

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
