---
title: login
parent: PCD
---
---
# 1. 브랜치 만들기
feat/login

# 2. local.properties에 url 입력 후 리빌드
리빌드 명령어
```
./gradlew clean build
```
참고: 캐시 지우기
```
./gradlew clean
```
# 3. AndroidManifest.xml 권한 추가

### 체크 할 점
테스트하시면서 데이터 형식 안 맞거나 에러 나면
모든 API 엔드포인트 연동은 완료되어 있고 정상 동작하니까 앱 쪽 API 통신/UI 바인딩 개발은 바로 진행해도 돼. 다만 스캔 결과는 현재 더미 데이터라서 어떤 URL이든 같은 결과가 나올 거야. ML 모델 연결이랑 VT API 키 설정이 끝나면 실제 결과가 나올 것

# Encrypted CookieJar로 재시작 후에도 로그인 로컬에서 유지

|                     | 인메모리              | EncryptedCookieJar     |
| ------------------- | ----------------- | ---------------------- |
| refresh_token 저장 위치 | 메모리 (RAM)         | 암호화된 SharedPreferences |
| 앱 재시작 후             | 쿠키 사라짐 → 재로그인     | 쿠키 유지 → 자동 로그인         |
| 로그아웃 시 쿠키 삭제        | 자동 (앱 끄면 어차피 사라짐) | 직접 `clear()` 호출 필요     |
| refresh 실패 시 쿠키 삭제  | 자동 (앱 끄면 어차피 사라짐) | 직접 `clear()` 호출 필요     |
