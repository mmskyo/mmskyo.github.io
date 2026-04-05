---
title: android
parent: computerscience
nav_order: 2
---
---

실무에서 진행하는 **체계적인 개발 순서 4단계**를 튜토리얼로 정리
---

## 1단계: 앱의 지도 그리기 (Navigation Graph 구성)

화면(UI)을 그리기 전에, 어떤 화면들이 존재하고 어떻게 이동할지 **경로(Route)**부터 정의해야 합니다.

- **Auth Graph (인증 그래프):** 스플래시(로딩) 화면 ➡️ 로그인 화면 ➡️ (필요시) 회원가입 화면
    
- **Main Graph (메인 그래프):** 대시보드(아까 보여주신 화면) ➡️ 상세 로그 화면 ➡️ 설정 화면
    
- **실무 포인트:** Compose의 `NavHost`를 사용해, 앱이 켜질 때 "자동 로그인이 되어있는가?"를 판단하여 Auth Graph로 갈지, Main Graph로 갈지 분기 처리하는 뼈대 코드를 먼저 작성합니다.
    

## 2단계: 데이터 저장소 및 상태 관리 세팅 (ViewModel & DataStore)

로그인 화면에서 아이디/비밀번호를 입력받으면 서버와 통신해야 하고, 성공하면 '토큰(Token)'을 저장해야 합니다.

- **토큰 저장소:** 안드로이드 실무에서는 기존의 SharedPreferences 대신 **Jetpack DataStore**를 사용해 로그인 토큰이나 유저 정보를 안전하게 저장합니다.
    
- **상태 관리:** UI와 데이터를 분리하기 위해 `LoginViewModel`을 만듭니다. 이 ViewModel이 "로그인 중(Loading)", "성공(Success)", "실패(Error)" 상태를 관리하도록 세팅합니다.
    

## 3단계: 공통 디자인 시스템 (Design System) 만들기

본격적으로 화면을 그리기 직전, 팀원들과 **공통으로 쓸 UI 부품**을 먼저 만듭니다.

- **이유:** 로그인 화면의 '확인 버튼'과 메인 화면의 '스캔 버튼'은 색상이나 모서리 둥글기가 같을 확률이 높습니다.
    
- **작업 내용:** `ui/theme` 폴더 안에 브랜드 색상, 타이포그래피(글꼴 크기), 공통 버튼(`CustomPrimaryButton`), 공통 텍스트 입력창(`CustomTextField`)을 미리 만들어 둡니다. 이렇게 하면 나중에 UI를 조립하듯 빠르게 만들 수 있습니다.
    

## 4단계: 화면(UI) 구현 (로그인 ➡️ 메인 대시보드)

이제 드디어 눈에 보이는 화면을 구현할 차례입니다! 만들어둔 뼈대와 부품을 결합합니다.

1. **로그인 화면 UI:** 3단계에서 만든 공통 텍스트 필드와 버튼을 배치합니다.
    
2. **동작 연결:** 버튼을 누르면 2단계에서 만든 `LoginViewModel`에 아이디/비번을 전달합니다.
    
3. **화면 이동:** 로그인이 성공하면 1단계에서 만든 `NavController`를 이용해 메인 대시보드 화면으로 슉! 넘어갑니다.
    
4. **메인 화면 UI 구현:** 아까 보여주신 카드형 대시보드와 하단 탭 바를 구현합니다.
    

---

> **💡 요약하자면?**
> 
> 실무에서는 **[경로 설정 ➡️ 데이터/상태 세팅 ➡️ 공통 부품 제작 ➡️ UI 조립]** 순서로 진행합니다. 이렇게 해야 나중에 코드가 꼬이지 않고 팀원들과 업무를 나누기(한 명은 로그인 UI, 한 명은 통신 세팅 등) 편합니다.

# Navigation Host 사용법
## 경로 정의 및 NavHostController 만들기

### 구성요소
- NavController: 대상(즉, 앱의 화면) 간 이동을 담당
- NavGraph: 이동할 컴포저블 대상을 매핑
- NavHost: NavGraph의 현재 대상을 표시하는 컨테이너 역할을 하는 컴포저블

### 앱에서 대상 경로 정의 
앱의 화면 수는 한정되어있으므로 경로도 한정됨
enum 클래스를 사용해서 경로 정의(속성 이름이 포함된 문자열을 반환하는 이름 속성이 있음)

먼저 앱의 경로를 정의 
예시) cupcake 앱
- start: 버튼 세개 중에서 원하는 수량 선택
- Flavor: 옵션 목록에서 맛을 선택
- Pickup: 옵션 목록에서 수령일 선택
- Summary: 선택한 내용 검토하고 주문 전송 or 취소
enum 클래스를 추가해 경로를 정의
1. CupcakeScreen.kt 에서 CupcakeAppBar 컴포저블 위에 enum 클래스 CupcakeScreen을 추가
```kotlin
enum class CupcakeScreen() {
}
```
2. enum 클래스에 4가지 사례 start, flavor, pickup, summary 추가
```kotlin
enum class CupcakeScreen() {
	Start,
	Flavor,
	Pickup,
	Summary
}
```
### 앱에 NavHost 추가
지정된 경로를 기반으로 다른 컴포저블 대상을 표시하는 컴포저블
경로가 flavor인 경우
navhost는 컵케이크맛 선택하는 화면을 표시
경로가 summary면 앱에는 요약화면
문법
```
NavHost(
	navController,
	startDestination,
	modifier,
) {
	content
}
```
- navController: NavHostController 클래스의 인스턴스, navigate() 메소드를 호출하여 다른 대상으로 이동

# 뷰 바인딩
findviewbyid 이제 안쓴다. ui 요소를 안전하고 편하게 가져오기 위해 view binding 사용
build.gradle.kts(module:app)에 설정 추가


# 초기 세팅
1. 리포지토리 클론
2. 안드로이드 스튜디오에서 뉴프로젝트를 클론한 파일 경로에 만들기
3. gitignore 수정
4. local properties 예시 파일 만들기 (그레이들 파일에도 추가)
5. 첫 커밋
6. develop 브랜치 생성

# 라이브러리 설정 ( 의존성 추가 )
libs.versions.toml 에서  라이브러리 추가 후 app/build.gradle.kts에서 실제 사용 선언

## libs.versions.toml

캐쉬 문제있을 때 invalidate caches

# build config field
defaultconfig 안에
```
buildConfigField("String", "BASE_URL", "\"https://localhost/\"")
//               타입      변수명       값
```
빌드할 때 코드안에 상수를 주입하는 함수

이렇게 하면 자동으로 gradle이 buildConfig.java파일을 생성
```
// 자동 생성되는 파일 (건드릴 필요 없음)
public final class BuildConfig {
    public static final String BASE_URL = "https://localhost/";
}
```
코드 어디서나 이렇게 사용
```
// NetworkModule.kt
Retrofit.Builder()
    .baseUrl(BuildConfig.BASE_URL)  // ← 여기서 사용
    .build()
```

```
buildConfigField(  
    "String", "BASE_URL",  
    "\"${localProperties.getProperty("BASE_URL") ?: "https://localhost/"}\""  
)
```
여기서 localhost는 fallback 기본값이다.
base url이 없을 때만 저것을 실행
**url을 하드코딩하면 안되기에 빌드컨픽을 쓴다**

# 커밋
첫 커밋
```
git add .
git commit -m "chore: Android 프로젝트 초기 세팅"
git push origin main
```

디벨롭 브랜치
```
git checkout -b develop
git push origin develop
```
피처 브랜치
```
git checkout develop
git checkout -b feature/android-login-ui
```

# 디렉토리 설정

## MVVM 아키텍처
```
사용자가 QR 스캔
    ↓
ui/         → 화면 표시 (Fragment, ViewModel)
    ↓
domain/     → 비즈니스 규칙 (어떤 데이터가 필요한지)
    ↓
data/       → 실제 데이터 가져오기 (서버 호출, DB)
    ↓
di/         → 위 레이어들을 연결해주는 접착제 (Hilt)
```

# 의존성 라이브러리
아, 표로 정리해달라는 거죠!

---

### 라이브러리 의존성 설명

|카테고리|라이브러리|버전|역할|
|---|---|---|---|
|**기본**|core-ktx|1.13.1|Kotlin Android 확장 함수 모음. `context.toast()` 같은 편의 기능 제공|
|**기본**|appcompat|1.7.0|구버전 Android 호환성 지원. Activity, Fragment 기본 기능|
|**기본**|material|1.12.0|Google Material Design 컴포넌트. 버튼, 카드, 다이얼로그 등|
|**기본**|activity|1.9.3|Activity 관련 최신 기능. `registerForActivityResult` 등|
|**기본**|constraintlayout|2.2.0|XML 레이아웃에서 뷰 위치를 유연하게 배치|
|**네트워크**|retrofit|2.11.0|서버 API 호출 라이브러리. `@GET`, `@POST` 어노테이션으로 API 정의|
|**네트워크**|retrofit-gson|2.11.0|Retrofit이 JSON 응답을 Kotlin 데이터 클래스로 자동 변환|
|**네트워크**|okhttp|4.12.0|Retrofit 아래에서 실제 HTTP 통신을 담당|
|**네트워크**|okhttp-logging|4.12.0|디버그 시 네트워크 요청/응답을 로그로 출력. 개발할 때 필수|
|**DI**|hilt-android|2.52|의존성 주입 프레임워크. `@Inject`로 객체를 자동으로 생성/주입|
|**DI**|hilt-compiler|2.52|Hilt가 컴파일 시 주입 코드를 자동 생성하기 위한 컴파일러|
|**로컬 DB**|room-runtime|2.6.1|Android 로컬 DB. 화이트리스트, 즐겨찾기 캐시 저장|
|**로컬 DB**|room-ktx|2.6.1|Room에 코루틴 지원 추가. `suspend fun` 으로 DB 조회 가능|
|**로컬 DB**|room-compiler|2.6.1|Room이 `@Dao` 어노테이션으로 DB 코드를 자동 생성하기 위한 컴파일러|
|**카메라**|camera-camera2|1.4.1|CameraX의 핵심. Camera2 API를 쉽게 사용할 수 있게 래핑|
|**카메라**|camera-lifecycle|1.4.1|카메라를 Activity/Fragment 생명주기에 맞게 자동 관리|
|**카메라**|camera-view|1.4.1|XML에서 `PreviewView`로 카메라 미리보기 화면 표시|
|**QR 스캔**|mlkit-barcode|17.3.0|Google ML Kit. 카메라 프레임에서 QR 코드를 실시간 인식/디코딩|
|**인증**|kakao-user|2.20.1|카카오 로그인 SDK. 카카오 계정으로 로그인 후 토큰 발급|
|**보안**|security-crypto|1.1.0-alpha06|JWT 토큰을 AES256으로 암호화해서 SharedPreferences에 저장|
|**브라우저**|browser|1.8.0|Custom Tabs. Safe URL을 앱 안에서 브라우저처럼 열기|
|**네비게이션**|navigation-fragment|2.8.5|Fragment 간 화면 이동 관리. `nav_graph.xml`로 흐름 정의|
|**네비게이션**|navigation-ui|2.8.5|BottomNavigationView와 NavController 연결|
|**라이프사이클**|lifecycle-viewmodel|2.8.7|ViewModel 제공. 화면 회전해도 데이터 유지|
|**라이프사이클**|lifecycle-runtime|2.8.7|`lifecycleScope`, `repeatOnLifecycle` 등 코루틴 생명주기 연동|
|**코루틴**|coroutines-android|1.9.0|비동기 처리. 서버 호출, DB 조회를 메인 스레드 차단 없이 처리|

---

### 컴파일러 계열 따로 정리

`ksp()`로 선언하는 것들은 런타임에 동작하는 게 아니라 **빌드 시 코드를 자동 생성**하는 도구

|선언 방식|라이브러리|하는 일|
|---|---|---|
|`ksp(libs.hilt.compiler)`|Hilt 컴파일러|`@HiltViewModel`, `@Inject` 어노테이션 처리해서 DI 코드 자동 생성|
|`ksp(libs.room.compiler)`|Room 컴파일러|`@Dao`, `@Entity` 어노테이션 처리해서 DB 접근 코드 자동 생성|

---

### 이 라이브러리들이 QR Shield에서 어떻게 연결되냐면

```
사용자가 QR 코드를 카메라에 비춤
    ↓
[camera-camera2 + camera-view]   카메라 프리뷰 표시
    ↓
[mlkit-barcode]                  QR 코드 인식 → URL 추출
    ↓
[coroutines]                     비동기로 서버 호출
    ↓
[retrofit + okhttp]              클라우드팀 API Gateway에 URL 전송
    ↓
[gson]                           JSON 응답 → ScanResult 데이터 클래스 변환
    ↓
[lifecycle-viewmodel]            결과를 ViewModel이 보관
    ↓
[navigation]                     결과 화면으로 이동
    ↓
[browser]                        Safe URL이면 Custom Tabs로 열기
    ↓
[room]                           Safe URL을 로컬 화이트리스트에 저장
    ↓
[security-crypto]                JWT 토큰 암호화 저장
    ↓
[hilt]                           위 모든 객체를 자동으로 연결
```

### 전체 화면 목록

| 화면              | Fragment 이름             | 설명                       |
| --------------- | ----------------------- | ------------------------ |
| 스플래시            | SplashFragment          | Q-Guard 로고 + 시스템 시작하기 버튼 |
| 로그인             | LoginFragment           | 이메일/비밀번호 로그인             |
| 회원가입            | RegisterFragment        | 이메일/비밀번호/닉네임 입력          |
| 스캔              | ScanFragment            | CameraX QR 스캔 화면 (메인)    |
| 보안 엔진 검사 중      | ScanningFragment        | 분석 중 로딩 화면               |
| 결과 - 안심 등록된 연결  | ResultWhitelistFragment | 화이트리스트 히트 시              |
| 결과 - 위험 차단됨     | ResultBlockedFragment   | BLOCKED 판정 시             |
| 결과 - 안전/검사 통과   | ResultSecuredFragment   | SECURED/SUSPICIOUS 판정 시  |
| 보안 로그 대시보드      | LogsFragment            | 스캔 이력 + 통계               |
| 안심 URL (화이트리스트) | WhitelistFragment       | 안심 URL 목록 관리             |

---

### Nav Graph 흐름

### nav_graph.xml에 넣을 Fragment 목록

```
SplashFragment          → 시작 화면
LoginFragment           → 로그인
RegisterFragment        → 회원가입
ScanFragment            → 메인 스캔 (바텀 탭)
ScanningFragment        → 분석 중 로딩
ResultWhitelistFragment → 안심 등록된 연결
ResultSecuredFragment   → SECURED / SUSPICIOUS 결과
ResultBlockedFragment   → BLOCKED 위험 차단
LogsFragment            → 보안 로그 대시보드 (바텀 탭)
WhitelistFragment       → 안심 URL 목록 (바텀 탭)
ProfileFragment         → 내 정보 (바텀 탭)
```

---

### 핵심 흐름 정리

**앱 시작 시** — 토큰 유효하면 Splash → Scan, 없으면 Splash → Login

**스캔 흐름** — 로컬 화이트리스트 히트 시 Scanning 건너뛰고 ResultWhitelist로 바로 이동, 아니면 Scanning → 결과 3갈래

**신고하기** — Logs 화면에서 특정 로그 탭 → 신고 다이얼로그 (별도 Fragment보다 Dialog 방식 추천)

**로그아웃** — Profile → 로그아웃 → JWT 삭제 → Login으로 popUpTo 전체 백스택 클리어

# 에러
---
## java heap space
: gradle이 빌드 작업을 하는데, 사용할 수 있는 메모리(램)이 꽉 찼다.
#### 이유
	1. 프로젝트 규모 확장: 코드가 늘어나거나 라이브러리를 추가하면서 빌드 시 필요한 메모리가 한계점을 넘음
	2. 캐시 누적: 빌드가 반복되면서 메모리 찌꺼기가 남을 때
	3. 이미지/리소스 추가: 고화질 이미지 많이 넣을 시 빌드할 때 메모리를 엄청 잡아먹음
#### 해결방안
1. gradle.properties 파일에서 메모리 사용량 늘리기
-Xmx4g  : 4기가바이트까지 메모리를 쓰겠다 (보통 512mb~1gb)
```
org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=512m
```
2. invalidate caches
현재 프로젝트의 빌드 찌꺼기만 삭제하는 명령어
```
./gradlew clean
```
cf. embedded ones까지 지우면 그레이들 파일까지 삭제되는 경우가 있음(경험담)
local history는 깃에 커밋을 했다면 안전


## DexMergingTaskDelegate
MultiDex 또는 라이브러리 충돌 문제
안드로이드는 내가 짠 코드를 DEX라는 파일로 변환해서 앱에 담는다.
그러나 담으려는 코드가 너무 커서 하나로 합치다가 보따리가 터지거나, 똑같은 이름의 코드가 두개나 있어서 뭘써야하는지 모르겠는 상황

이유: 
	1. 64K 메소드 제한: 안드로이드 앱 하나가 가질 수 있는 함수의 개수는 65,536개(64K)로 제한되어 있습니다. 라이브러리를 많이 추가하다 보면 이 한도를 넘어서는데, 이때 "합치기(Merge) 실패"가 뜹니다.
	2. 중복 라이브러리: 서로 다른 라이브러리들이 똑같은 외부 기능을 포함하고 있어서, 합칠 때 "어? 이거 아까 넣었는데 또 있네?" 하고 충돌이 나는 경우입니다.

해결방안
1. MultiDex 활성화 : 코드의 양이 많은게 문제라면, 보따리를 여러개 써도된다고 허락
```
android {
    defaultConfig {
        ...
        multiDexEnabled true  // 이 줄을 추가!
    }
} // minsdkversion21이상
```
2. 캐시문제(내 경우)
3. 라이브러리 확인 : 새로 추가한 implementation 확인하기


# 핵심 코드 "왜" 썼나요?
① Hilt (@AndroidEntryPoint, @Inject)
•
이유: 필요한 도구를 내가 직접 만들지 않고, "누가 이 가방 좀 가져다줘!" 하면 안드로이드가 알아서 챙겨주는 시스템이에요. 나중에 도구를 바꾸기 정말 편해집니다.
② StateFlow (uiState)
•
이유: 화면이 TV라면, StateFlow는 방송 신호예요. 화면은 가만히 있다가 신호가 바뀔 때만(Success -> Error) 딱 그 부분만 다시 그려요. 배터리와 메모리를 아끼는 일등 공신입니다.
③ runTest 와 Dispatcher.setMain
•
이유: 테스트 환경은 '시간이 멈춘 방' 같아요. runTest는 멈춘 시간을 빠르게 감기 해주고, setMain은 폰에만 있는 '메인 도로'를 테스트용 '임시 도로'로 교체해 주는 작업이에요.

# 앱 3층 구조
| 층 layer    | 이름                  | 하는 일                                                   |
| ---------- | ------------------- | ------------------------------------------------------ |
| 3층-ui      | fragment, viewmodel | 사용자 눈에 보이는 곳, 사용자가 버튼을 누르면 서버에 물어보라고 시키고 결과가 오면 화면을 바꿈 |
| 2층-do main | model, repository   | 앱의 두뇌, 서버에서 가져온 복잡한 데이터를 앱이 쓰기 좋게 가공(에러 메시지 번역도)       |
| 1층-data    | apiservice, room    | 실제 일꾼, 진짜 서버에 편지를 보내거나(retrofit), 폰 안에 데이터를 저장(room)   |
왜 내 눈엔 SharedPreferences가 안 보일까요?
코드를 보면 EncryptedSharedPreferences.create(...)라는 함수를 쓰고 있죠?
•
비밀: EncryptedSharedPreferences는 사실 일반 SharedPreferences를 **'상속'**받아서 만든 거예요. 즉, 겉모습(이름)은 Encrypted...이지만, 내부적으로는 안드로이드의 기본 저장소 시스템인 SharedPreferences를 사용하면서 데이터만 암호화해서 넣는 방식입니다.
•
그래서 prefs.edit().putString(...) 같은 익숙한 코드들이 보이는 거예요!
2. 우리가 이미 암호화 완료한 곳 (2군데)
3. TokenManager.kt: JWT 액세스 토큰을 저장하는 곳. (EncryptedSharedPreferences 사용 중)
4. EncryptedCookieJar.kt: 서버에서 내려준 refresh_token 쿠키를 저장하는 곳. (EncryptedSharedPreferences 사용 중)
5. NetworkModule.kt가 하는 일
보여주신 NetworkModule.kt는 이 두 가지 저장소를 **'연결'**해주는 정거장 역할을 해요.
•
provideCookieJar(encryptedCookieJar: EncryptedCookieJar) 함수를 통해, 아까 우리가 만든 암호화된 쿠키 저장소를 실제 네트워크 통신(OkHttpClient)에 합체시켜 줍니다.
•
결과적으로, 모든 통신 데이터(토큰, 쿠키)가 암호화되어 저장되도록 아주 튼튼하게 설계되어 있습니다.

IOException 더 구체적으로 나누어서 종류도 파악
```
private fun parseThrowable(e: Throwable) = when (e) {
    // 1. 길을 못 찾을 때 (UnknownHostException, ConnectException)
    is java.net.UnknownHostException, 
    is java.net.ConnectException -> "인터넷 연결이 불안정합니다"

    // 2. 상대방이 너무 늦게 대답할 때 (SocketTimeoutException)
    is java.net.SocketTimeoutException -> "요청 시간이 초과됐습니다"

    // 3. 그 외의 모든 상황 (Exception)
    else -> e.message ?: "알 수 없는 오류가 발생했습니다"
}
```
네트워크 인풋 아웃풋

비유로 설명하자면: 컴퓨터 입장에서 네트워크는 아주 멀리 떨어져 있는 **'외부 창고'**와 같음
1. 내가 서버에 편지를 보내는 것 자체가 **'나가는 작업(Output)'**
2. 서버에서 답장을 받는 것이 **'들어오는 작업(Input)'**
그런데 이 '편지 배달(IO)' 과정에서 배달원이 사고가 나거나(인터넷 끊김), 창고 문이 잠겨 있으면(서버 터짐) 배달 작업 자체가 실패하겠죠? 자바/코틀린에서는 이렇게 **"무언가를 주고받는 작업 중에 생기는 모든 사고"**를 통칭해서 **IOException**이라고 부릅니다.
그래서 네트워크 문제(SocketTimeoutException, UnknownHostException 등)는 모두 IOException의 자식들인 거예요!

# on create view
return binding.root 뒤에는 죽은코드! 나 환경 설정 다했으니까 이제 가져가

```kotlin
package com.qguarder.android.ui.favorites

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment
import androidx.fragment.app.viewModels
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import com.qguarder.android.databinding.FragmentFavoritesBinding
import dagger.hilt.android.AndroidEntryPoint
import kotlinx.coroutines.launch

@AndroidEntryPoint
class FavoritesFragment : Fragment() {

    private var _binding: FragmentFavoritesBinding? = null
    private val binding get() = _binding!!
    private val viewModel: FavoritesViewModel by viewModels()

    // 1. 여기서 건물의 뼈대를 세웁니다.
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View {
        _binding = FragmentFavoritesBinding.inflate(inflater, container, false)
        return binding.root
    }

    // 2. 건물이 완공된 후, 여기서 가구를 배치(로직 구현)합니다.
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // 로직은 무조건 여기서! (onViewCreated)
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.favorites.collect { list ->
                    if (list.isEmpty()) {
                        binding.rvFavorites.visibility = View.GONE
                        binding.layoutEmpty.visibility = View.VISIBLE
                    } else {
                        binding.rvFavorites.visibility = View.VISIBLE
                        binding.layoutEmpty.visibility = View.GONE
                        // adapter.submitList(list) <- 나중에 리스트 연결할 곳
                    }
                }
            }
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

1.
collect는 '빨대'와 같아요: 수도꼭지(Flow)에 빨대를 꽂고 물이 나올 때까지 계속 기다리는 작업입니다. 그래서 suspend(멈춤 가능) 기능이 필요해요.
2.
lifecycleScope.launch는 '전용 일꾼'이에요: onViewCreated는 비동기 작업이 끝날 때까지 기다려주지 않고 바로 자기 할 일을 끝내버립니다. 그래서 **"나는 내 갈 길 갈 테니, 너(일꾼)는 옆에서 따로 빨대 꽂고 기다려!"**라고 시키는 것이 바로 launch입니다.
3.
repeatOnLifecycle은 '안전장치'예요: 사용자가 앱을 잠시 내렸을 때(홈 버튼 등)도 계속 물을 받으면 배터리가 낭비되겠죠? 이 코드는 화면이 보일 때만 빨대를 꽂고, 안 보이면 잠시 빼두는 똑똑한 역할을 합니다.

# logs view model
서버에서 보안 로그를 가져옴
```
package com.qguarder.android.ui.logs

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.qguarder.android.data.remote.ApiService
import com.qguarder.android.data.remote.dto.ScanLogItemDto
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class LogsViewModel @Inject constructor(
    private val apiService: ApiService // 서버와 통신할 일꾼 주입
) : ViewModel() {

    // 로그 목록을 담는 통 (처음엔 빈 리스트)
    private val _logs = MutableStateFlow<List<ScanLogItemDto>>(emptyList())
    val logs = _logs.asStateFlow()

    init {
        loadLogs() // 뷰모델이 만들어지자마자 로그를 불러옵니다.
    }

    fun loadLogs() {
        viewModelScope.launch {
            try {
                val response = apiService.getScanLogs()
                if (response.isSuccessful) {
                    // 서버에서 받은 로그 리스트를 통에 담습니다.
                    _logs.value = response.body()?.logs ?: emptyList()
                }
            } catch (e: Exception) {
                // 에러가 나면 빈 리스트로 둡니다.
            }
        }
    }
}
```

# logs fragment 
데이터가 비어있는지 확인해서 화면을 바꿔줌
```
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    // 로직은 무조건 onViewCreated에서!
    viewLifecycleOwner.lifecycleScope.launch {
        viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
            // ViewModel의 로그 목록을 지켜봅니다.
            viewModel.logs.collect { list ->
                if (list.isEmpty()) {
                    // 로그가 없으면 리스트를 숨기고 '비어있음' 화면을 보여줍니다.
                    binding.rvLogs.visibility = View.GONE
                    binding.layoutEmpty.visibility = View.VISIBLE
                } else {
                    // 로그가 있으면 리스트를 보여주고 '비어있음' 화면을 숨깁니다.
                    binding.rvLogs.visibility = View.VISIBLE
                    binding.layoutEmpty.visibility = View.GONE
                    // 나중에 여기에 리스트 어댑터 연결 코드가 들어갑니다!
                }
            }
        }
    }
}
```

복습 포인트
}
1.
왜 init에 loadLogs()를 넣었나요?: 화면이 켜지자마자 로그를 보여주고 싶으니까, 뷰모델이 태어나자마자 일을 시키는 거예요.
2.
왜 collect를 쓰나요?: 서버에서 데이터를 가져오는 데 시간이 걸리죠? collect는 데이터가 도착할 때까지 기다렸다가, 데이터가 들어오는 순간 "어! 데이터 왔다! 화면 바꿔!"라고 실행해주는 아주 똑똑한 친구예요.
3.
repeatOnLifecycle은 왜 썼나요?: 사용자가 앱을 잠시 내리고 유튜브를 보고 있을 때는 화면을 그릴 필요가 없죠? 그때 잠시 관찰을 멈춰서 배터리를 아끼기 위함입니다.