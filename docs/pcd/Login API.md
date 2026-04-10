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

# 토큰 갱신 문제
1. **AuthInterceptor가 일시적 실패에도 모든 토큰을 삭제** — 네트워크 에러로 refresh가 실패해도 `tokenManager.clear()` + `cookieJar.clear()`를 호출해서 refresh_token 쿠키까지 날려버림. 다음 앱 실행 시 복구 불가.
    
2. **앱 시작 시 refresh 시도 없음** — access_token이 (위 이유로) 없어지면, refresh_token 쿠키가 살아있어도 바로 로그인 화면으로 보냄.
    

플랜은 5개 파일 수정으로 구성됩니다. 핵심은:

- AuthInterceptor에서 네트워크 에러 시 refresh cookie를 보존
- SplashFragment에서 access_token이 없어도 refresh cookie가 있으면 refresh 시도 후 자동 로그인

### 1. `TokenManager.kt` — `clearAccessToken()` 추가

```kotlin
/** access_token만 삭제 (일시적 refresh 실패 시) */
fun clearAccessToken() {
    prefs.edit().remove(KEY_ACCESS).apply()
}
```

---

### 2. `EncryptedCookieJar.kt` — `hasRefreshCookie()` 추가

```kotlin
/** refresh_token 쿠키가 저장되어 있는지 확인 */
fun hasRefreshCookie(): Boolean {
    for ((_, cookies) in memoryCache) {
        if (cookies.any { it.name == "refresh_token" }) return true
    }
    val allPrefs = prefs.all
    for ((key, value) in allPrefs) {
        if (key.startsWith(KEY_COOKIES) && value is String) {
            try {
                val jsonArray = JSONArray(value)
                for (i in 0 until jsonArray.length()) {
                    if (jsonArray.getJSONObject(i).getString("name") == "refresh_token") {
                        return true
                    }
                }
            } catch (_: Exception) {}
        }
    }
    return false
}
```

---

### 3. `AuthInterceptor.kt` — 401 핸들링 전체 수정

```kotlin
override fun intercept(chain: Interceptor.Chain): Response {
    val request = chain.request()
    if (noAuthPaths.any { request.url.encodedPath.contains(it) }) {
        return chain.proceed(request)
    }

    val authed = request.newBuilder()
        .header("Authorization", "Bearer ${tokenManager.getAccessToken()}")
        .build()
    val response = chain.proceed(authed)

    if (response.code == 401) {
        response.close()
        val refreshResponse = runBlocking {
            try {
                RefreshApiServiceHolder.service.refreshToken()
            } catch (e: Exception) {
                null // 네트워크 에러
            }
        }

        when {
            // 네트워크 에러 — refresh cookie 보존
            refreshResponse == null -> {
                tokenManager.clearAccessToken()
                return response
            }
            // refresh 성공
            refreshResponse.isSuccessful -> {
                val newToken = refreshResponse.body()?.accessToken
                if (newToken != null) {
                    tokenManager.updateAccessToken(newToken)
                    return chain.proceed(
                        request.newBuilder()
                            .header("Authorization", "Bearer $newToken")
                            .build()
                    )
                }
                tokenManager.clearAccessToken()
                return response
            }
            // 서버가 refresh token 명확히 거부 — 전체 삭제
            refreshResponse.code() in listOf(401, 403) -> {
                tokenManager.clear()
                cookieJar.clear()
                return response
            }
            // 5xx 등 서버 에러 — refresh cookie 보존
            else -> {
                tokenManager.clearAccessToken()
                return response
            }
        }
    }
    return response
}
```

---

### 4. `AuthRepository.kt` — `tryRefreshSession()` 추가

```kotlin
/** 앱 시작 시 refresh cookie로 세션 복구 시도 */
suspend fun tryRefreshSession(): Boolean {
    return try {
        val response = RefreshApiServiceHolder.service.refreshToken()
        if (response.isSuccessful) {
            val token = response.body()?.accessToken
            if (token != null) {
                tokenManager.updateAccessToken(token)
                true
            } else {
                false
            }
        } else if (response.code() in listOf(401, 403)) {
            tokenManager.clear()
            cookieJar.clear()
            false
        } else {
            false
        }
    } catch (e: Exception) {
        false
    }
}
```

---

### 5. `SplashFragment.kt` — startup refresh 연결

import 추가:

```kotlin
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.launch
import com.qguarder.android.data.local.EncryptedCookieJar
import com.qguarder.android.data.remote.AuthRepository
```

주입 추가:

```kotlin
@Inject lateinit var tokenManager: TokenManager
@Inject lateinit var authRepository: AuthRepository
@Inject lateinit var cookieJar: EncryptedCookieJar
```

버튼 클릭의 `withEndAction` 블록 변경:

```kotlin
.withEndAction {
    if (tokenManager.isLoggedIn()) {
        findNavController().navigate(R.id.action_splash_to_scan)
    } else if (cookieJar.hasRefreshCookie()) {
        binding.btnStart.isEnabled = false
        lifecycleScope.launch {
            val refreshed = authRepository.tryRefreshSession()
            if (refreshed) {
                findNavController().navigate(R.id.action_splash_to_scan)
            } else {
                findNavController().navigate(R.id.action_splash_to_login)
            }
        }
    } else {
        findNavController().navigate(R.id.action_splash_to_login)
    }
}
```

---

# 앱 재실행 시 로그아웃되는 버그 수정

## Context

앱을 시간이 지난 후 다시 열면 로그인이 풀려있는 문제.

**근본 원인**: `EncryptedCookieJar.saveFromResponse()`가 쿠키를 **병합하지 않고 교체**함.

- 로그인 시 `refresh_token` 쿠키가 저장됨

- 이후 다른 API 응답에 `Set-Cookie` 헤더가 오면 (세션 쿠키 등) → 기존 `refresh_token` 쿠키가 통째로 덮어써짐

- access_token 만료 → refresh 시도 → refresh_token 쿠키 없음 → 실패 → 전체 토큰 삭제 → 로그아웃

## 수정 파일 (1개)

### `EncryptedCookieJar.kt`

- `app/src/main/java/com/qguarder/android/data/local/EncryptedCookieJar.kt`

`saveFromResponse()`를 수정: 쿠키를 **교체(replace)** → **병합(merge)** 방식으로 변경

**변경 전** (라인 45-68):

```kotlin

override fun saveFromResponse(url: HttpUrl, cookies: List<Cookie>) {

if (cookies.isEmpty()) return

memoryCache[url.host] = cookies // 전체 교체!

// ... prefs에도 전체 교체

}

```

**변경 후**:

```kotlin

override fun saveFromResponse(url: HttpUrl, cookies: List<Cookie>) {

if (cookies.isEmpty()) return

// 기존 쿠키 로드 후 병합 (name+domain+path 기준)

val existing = loadForRequest(url).associateBy { Triple(it.name, it.domain, it.path) }.toMutableMap()

for (cookie in cookies) {

existing[Triple(cookie.name, cookie.domain, cookie.path)] = cookie

}

val merged = existing.values.toList()

memoryCache[url.host] = merged

val jsonArray = JSONArray()

merged.forEach { cookie ->

val obj = JSONObject().apply {

put("name", cookie.name)

put("value", cookie.value)

put("domain", cookie.domain)

put("path", cookie.path)

put("expiresAt", cookie.expiresAt)

put("secure", cookie.secure)

put("httpOnly", cookie.httpOnly)

}

jsonArray.put(obj)

}

prefs.edit()

.putString("${KEY_COOKIES}_${url.host}", jsonArray.toString())

.apply()

}

```

핵심: `name + domain + path`가 같은 쿠키는 업데이트하고, 새로 온 적 없는 기존 쿠키는 그대로 유지.

## 검증

- 빌드 확인 (`./gradlew assembleDebug`)

- 시나리오 테스트:

1. 로그인 후 다른 API 호출 → refresh_token 쿠키가 유지되는지 확인

2. 시간이 지난 후 앱 재실행 → 자동 로그인 유지되는지 확인

3. 로그아웃 → `clear()` 호출 시 모든 쿠키 삭제되는지 확인
