---
title: Backend
nav_order: 2
parent: Projects
---
---

# FastAPI
- fastapi 첫걸음 https://fastapi.tiangolo.com/ko/tutorial/first-steps/

api 사용 시퀀스 정리 예) 스캔->데이터전송->비교

- api 명세서 작성법: https://cobinding.tistory.com/165

- 엔드포인트: https://evan-moon.github.io/2020/04/07/about-restful-api/


# 코틀린
- https://velog.io/@harperkwon/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%BD%94%EB%93%9C-%ED%92%88%EC%A7%88%EC%9D%98-%EC%8B%9C%EC%9E%91-ktlint-%EC%99%84%EB%B2%BD-%EC%A0%95%EB%B3%B5-%EA%B0%80%EC%9D%B4%EB%93%9C-%EC%84%A4%EC%A0%95%EB%B6%80%ED%84%B0-%EC%8B%A4%EC%A0%84-%ED%99%9C%EC%9A%A9%EA%B9%8C%EC%A7%80 코틀린 품질
- 안드로이드 개발자 https://pluu.github.io/blog/owner/2020/12/26/start-android-developer/
- documentation https://kotlinlang.org/docs/home.html

# DB
- orm https://eun-jeong.tistory.com/31
- 

인사이트 참고 사이트 surfit.io

# 백-서버
서버 요청이 많은 retrofit+polling(폴링) 방식이 apiGateway때문에 정상 사용자도 블락되는거 아니냐?
-> 폴링 간격 늘리기, 상태 변경시만 조회, redis로 결과캐싱(빠른 응답), websocket or sse 해결 가능
celery 혼자 못돌아가기에 중간 전달자가 필요 -> redis
아무튼 오래걸리니까 오래걸리는 작업을 백그라운드에서 처리할 수 있게 셀러리를 사용
우리는 이미 결과 캐싱=결과 저장, 조회 최적화 를 쓰지만, 작업을 분리했느냐 는 다른 문제

## 캐싱 (redis)
```
GET /scan/abc123 -> redis에서 결과 가져옴
```

이미 계산된 결과를 빠르게 줌
### 비동기 처리 (celery, lambda)
```
POST /scan -> 작업 등록 -> 나중에 처리
```
오래 걸리는 작업을 따로 돌림

따라서 redis만 있으면 충분하지 않고 작업을 따로 돌리는 구조가 있어야한다.

## 셀러리를 안쓴다면
서버리스 구조임
lambda가 셀러리 대체 가능
이벤트 기반 실행

셀러리는 
FastAPI → Redis → Celery Worker → 처리

람다는
API Gateway → Lambda → (비동기 처리)
또는
FastAPI → Lambda 호출 → 작업 처리

따라서 타임아웃에 좋은 구조는
```
POST /scan → Lambda 비동기 호출 → 바로 응답
```
나쁜구조는 
```
POST /scan → Lambda 끝날 때까지 기다림
```
여전히 타임 아웃 발생

폴링 간격 조절 5-10초
상태기반 폴링
processing → 5초 간격
almost_done → 2초 간격
결과 나오면 종료 - 불필요한 요청 제거
고급) 웹소켓/sse - gateway제한 영향 거의 없음

# 상태 코드
http 상태 코드 - 요청 자체 성공/실패 ex. 200
바디에 있는 status - 작업 진행 상태 ex. processing

# 4주차 클라우드 질문
1. api gateway 호출 제한
2. 상태값 정의 init(콜드스타트시 프리워밍), queued, analyzing_ml, analyzing_vt, completed, failed 이렇게 괜찮을까요?
3. 스캔 검사 기능은 콜드 스타트나 ml분석때문에 시간이 좀 걸릴까봐 응답에 상태값 포함하고싶은데 추가해도 괜찮을까요?
4. lambda 함수별 타임아웃 설정
5. lambda 비동기 처리 방식