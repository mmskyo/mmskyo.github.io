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
뒤에서 돌고있는 빌드 엔진들 gradle daemon 다 끄기
```
./gradlew --stop
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

- [ ]  저장소를 영구적으로 바꿨나?
- [ ]  `tokenManager.clear()` 가 호출되는 곳 전부 찾았나? (`Grep: tokenManager.clear`)
- [ ]  그 모든 곳에 `cookieJar.clear()` 도 추가했나?
- [ ]  새로 만든 클래스가 실제로 DI에 연결됐나? (`NetworkModule` 확인)

# 테스트 모듈 만들기
테스트하고싶은 파일을 열고(loginviewmodel) 클래스 이름에 커서를 올려놓고 option+enter(맥)/alt+enter(win) 를 누름
![[Screenshot 2026-03-31 at 12.09.32 PM.png|323]]해당하는 라이브러리 JUnit4 선택 후 fix,
이름과 목적지는 그대로
setUp/@Before 선택 -> 테스트 전 가짜 서버나 뷰모델을 준비하는 코드 자동생성
generate test methods for: 테스트하고싶은 함수 선택 -> login()
getuistate는 안드로이드테스트(에뮬레이터사용, 무거움, 화면 확인)로 할거임
ok누르면 생성할 폴더 선택하고 생성된 파일에 테스트 코드 작성

## Given-When-Then 공식:
Given: 이런 상황일 때 (가짜 데이터 준비)
When: 이걸 하면 (함수 호출)
Then: 이렇게 되어야 한다 (결과 확인)

view model test
할때는 실행할 핸드폰이 없기에 
view model은 화면을 그리기 위해 main 쓰레드라는 특별한 길을 사용
꼭 가짜 메인스레드를 만들어줘야함
```
import kotlinx.coroutines.test.UnconfinedTestDispatcher
import org.junit.After

@OptIn(ExperimentalCoroutinesApi::class)
class LoginViewModelTesst {
	private val testDispatcher = UnconfinedTestDispatcher() // standard 는 그냥, unconfined는 코루틴은 비동기 함수이기에 기다리지않고 바로 실행할수 있도록(즉시 실행) 아니면 기다리라는 함수도 따로 사용가능
	
	@Before
	fun setUp() {
		Dispatchers.setMain(testDispatcher) // 메인스레드 설정
	}
	
	@After
	fun tearDown() {
		Dispatchers.resetMain() // 끝나고 가짜 스레드 지우기
	}
	
}
```