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
`