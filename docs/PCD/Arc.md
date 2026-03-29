---
title: architecture
nav_order: 3
parent: PCD
published: "true"
---
---
# 구현 순서

Fragment ( 화면 ) --- 사용자 입력 받음 ---> ViewModel (두뇌) --- 로직 처리 ---> Repository (창고지기) --- 서버 또는 DB 요청 ---> ApiService / Dao --- 실제 데이터 ---> 다시 ViewModel로 결과 전달 ---> Fragment에서 화면 업데이트

# 개발 순서
1. 데이터 모델 (Domain)
2. 데이터 소스 (Data)
3. 비즈니스 로직 (ViewModel)
4. 화면 (Fragment + XML)

## 1. 데이터모델 ( 도메인 )
즐겨찾기는 뭘 담고 있어야하지? 를 먼저 생각한다.
```
// domain/model/Models.kt
data class Bookmark(
	val bookmarkId: String, // 고유 아이디
	val url: String, // url
	val titleL String?, // 제목, null이어도됨?
	val riskLevel: RiskLevel, // 안전 판정
	val createdAt: String, // 등록 시간
	
)
```
## 2. DTO 정의
서버가 어떤 형태로 데이터를 줄까? - api 명세서를 보며 작성
```
// data/remote/dto/Dtos.kt
data class AddBookmarkRequest(
	val url: String,
	val title: String?,
)

data class BookmarksResponse(
	val bookmarks: List<BookmarkDto>
)

// 데이터모델도 있는데 여기서 또 정의하는 이유
// 둘을 분리하면 서버가 api를 바꿔도 dto만 수정하면 됨
// 예) 서버가 bookmark_id 에서 id로 바꿨을 경우
data class BookmarkDto(
	@SerializedName("bookmark_id") val bookmarkId: String,
	val url: String,
	val title: String?,
	@SerializedName("risk_level") val riskLevel: String,
	@SerializedName("created_at") val createdAt: String,
)
```

#### dto가 뭔데?
```
DTO = Data Transfer Object
      데이터 전달 객체

서버 ↔ 앱 사이에서
데이터를 주고받을 때 쓰는 형식이에요

서버가 주는 JSON
{
  "bookmark_id": "abc123",
  "risk_level": "SECURED"
}

이걸 받아서 저장하는 그릇이 DTO예요
data class BookmarkDto(
    @SerializedName("bookmark_id") val bookmarkId: String,
    @SerializedName("risk_level")  val riskLevel: String,
)
```

#### @SerializedName이 뭔데?
```
서버 JSON 키 이름과 코틀린 변수명이 다를 때 연결해줌

서버가 주는 것          코틀린에서 쓰고 싶은 이름
"bookmark_id"    →     bookmarkId
"risk_level"     →     riskLevel

@SerializedName("bookmark_id") val bookmarkId: String
          ↑                              ↑
    서버 JSON 키                   코틀린 변수명
```
	- 직렬화 병렬화는?
```
직렬화 (Serialization)
→ 코틀린 객체를 JSON 문자열로 변환
→ 서버에 보낼 때

Bookmark(url = "https://example.com")
         ↓ 직렬화
{"url": "https://example.com"}

역직렬화 (Deserialization)
→ JSON 문자열을 코틀린 객체로 변환
→ 서버에서 받을 때

{"bookmark_id": "abc", "risk_level": "SECURED"}
         ↓ 역직렬화
BookmarkDto(bookmarkId = "abc", riskLevel = "SECURED")

Gson 라이브러리가 이걸 자동으로 해줘요
@SerializedName이 연결고리 역할
```
## 3. ApiService에 엔드포인트 추가
서버 어디에 요청할까?
```kotlin
// data/remote/ApiService.kt
@GET("api/v1/users/me/bookmarks")
suspend fun getBookmarks(): Response<BookmarksResponse>

@POST("api/v1/users/me/bookmarks")
suspend fun addBookmark(
	@Body request: AddBookmarkRequest
): Response<BookmarkDto>

@DELETE("api/v1/users/me/bookmarks/{id}")
suspend fun deleteBookmark(
	@Path("id") bookmarkId: String
): Response<Unit>
```

질문
```
// 일반 함수
fun getBookmarks() { }
// → 호출하면 끝날 때까지 화면이 멈춰요

// suspend 함수
suspend fun getBookmarks() { }
// → 서버 응답 기다리는 동안
//   다른 작업 할 수 있어요 (화면 안 멈춤)
// → viewModelScope.launch 안에서만 호출 가능


// @ = "이거 특별한 역할 해줘" 라고 표시하는 것

@GET         서버에 GET 요청
@POST        서버에 POST 요청
@DELETE      서버에 DELETE 요청
@Body        요청 본문에 넣어
@Path        URL 경로에 넣어
@Query       URL 뒤에 ?key=value 형식으로 넣어
@Inject      이거 자동으로 주입해줘
@Singleton   이 클래스 하나만 만들어줘
@HiltViewModel Hilt가 이 ViewModel 관리해줘
@AndroidEntryPoint Hilt가 이 Fragment 관리해줘

// 쿼리 형식은 항상 저 코드대로?
// 형식은 API 명세서 보고 그대로 따라 쓰면됨(명세서에 get이면 get)

// response가 뭐야?
Response<BookmarksResponse>

Response = 서버 응답 전체 (상태코드 + 바디)
<>       = 안에 뭐가 들어있는지 표시

Response<BookmarksResponse>
→ 서버 응답 안에 BookmarksResponse가 들어있어

response.isSuccessful  → 200번대인지 확인
response.body()        → 실제 데이터 꺼내기
response.code()        → 상태 코드 (200, 404, 500...)

// @PATH, @BODY가 뭐임?

// @Path — URL 경로 안에 값 넣기
@DELETE("api/v1/bookmarks/{id}")
suspend fun delete(@Path("id") bookmarkId: String)
// bookmarkId = "abc123" 이면
// → api/v1/bookmarks/abc123 으로 요청

// @Body — 요청 본문에 객체 넣기
@POST("api/v1/bookmarks")
suspend fun add(@Body request: AddBookmarkRequest)
// request를 JSON으로 변환해서 본문에 담아 보냄
// → {"url": "https://example.com"}

// @Query — URL 뒤에 파라미터 붙이기
@GET("api/v1/scan-logs")
suspend fun getLogs(@Query("page") page: Int)
// → api/v1/scan-logs?page=1
```

## 4단계 - Repository 작성
서버 응답을 앱에서 쓸 수 있게 변환해줘.
```kotlin
// data/remote/BookmarkRepository.kt
@Singleton
class BookmarkRepository @Inject constructor(
	private val apiService: ApiService,
	private val favoriteCacheDao: FavoriteCacheDao,
) {
	// 즐겨찾기 목록 가져오기
	suspend fun getBookmarks(): Result<List<Bookmark>> {
		return try {
			val response = apiService.getBookmarks()
			if (!response.isSuccessful)
				return Result.failure(Exception("서버 오류"))
			}
			// DTO -> 도메인 모델 변환
			val bookmarks = response.body()!!.bookmarks.mapf {
				Bookmark(
					bookmarkId = it.bookmarkId,
					url = it.url,
					title = it.title,
					riskLevel = RiskLevel.from(it.riskLevel),
					createdAt = it.createdAt,
				)
			}
			Result.success(bookmarks)
		} catch (e: Exception) {
			Result.failure(e)
		}
	}
	
	// 즐겨찾기 추가
	suspend fun addBookmark(url: String): Result<Unit> {
		return try {
			val response = apiService.addBookmark(
				AddBookmarkRequest(url = url, title = null)
			)
			if (!response.isSuccessful) {
				return Result.failure(Exception("추가 실패"))
			}
			Result.success(Unit)
		} catch (e: Exception) {
			Result.Failure(e)
		}
	}
}
```

```
4. @Singleton / @Inject / try-catch / Unit

@Singleton이 뭐야?
싱글톤 = 앱 전체에서 딱 하나만 존재하는 객체

@Singleton
class BookmarkRepository

→ 앱 어디서든 BookmarkRepository를 요청하면
  항상 같은 하나의 인스턴스를 줌

왜 필요해?
Repository, ViewModel, TokenManager 같은 건 여러 개 만들 필요가 없음
하나만 있으면 충분하고
여러 개 만들면 데이터가 따로따로 관리돼서 문제생김

@Inject constructor가 뭐지?

class BookmarkRepository @Inject constructor(
    private val apiService: ApiService,
)

// @Inject constructor = "내가 필요한 재료를 Hilt가 자동으로 줘"

// 일반적으로는 이렇게 직접 만들어야 해요
val repo = BookmarkRepository(apiService = ApiService())

// @Inject constructor 붙이면
// Hilt가 알아서 만들어줘요
// ApiService도 알아서 찾아서 넣어줘요

// @Inject constructor = "Hilt야 나 만들어줘"

class BookmarkRepository @Inject constructor(
    private val apiService: ApiService,
)

// 이렇게 선언하면
// BookmarkRepository가 필요한 곳에서
// Hilt가 자동으로 만들어서 넣어줘요

// ApiService도 어딘가에 @Inject constructor 있으면
// Hilt가 ApiService도 자동으로 만들어서 넣어줘요

// 도미노처럼 연결돼요
FavoritesViewModel
    → BookmarkRepository  (@Inject constructor)
        → ApiService      (@Inject constructor)
            → OkHttpClient (NetworkModule에서 제공)

hilt가 뭐냐?
앱을 만들다 보면
클래스들이 서로를 필요로 해요

FavoritesViewModel이 필요한 것
→ BookmarkRepository

BookmarkRepository가 필요한 것
→ ApiService

ApiService가 필요한 것
→ OkHttpClient
→ Retrofit

이걸 직접 만들면 이렇게 돼요
val okHttp = OkHttpClient()
val retrofit = Retrofit(okHttp)
val apiService = ApiService(retrofit)
val repo = BookmarkRepository(apiService)
val viewModel = FavoritesViewModel(repo)

너무 복잡하죠?

Hilt가 이걸 전부 자동으로 해줘요
"내가 필요해" 라고 선언만 하면
Hilt가 알아서 만들어서 넣어줘요

요약
Hilt 클래스 자동으로 만들고 연결해주는 도구 
@Inject "Hilt야 나 만들어줘" 
constructor 생성할 때 필요한 재료 목록

return try catch문 왜?

return try {
    val response = apiService.getBookmarks()
    Result.success(bookmarks)
} catch (e: Exception) {
    Result.failure(e)
}

// 서버 요청은 언제든 실패할 수 있어요
// 인터넷 끊김, 서버 다운, 타임아웃...

// try 안에서 에러 나면
// catch가 잡아서 Result.failure로 반환
// 앱이 그냥 죽지 않아요

// try-catch 없으면
// 에러 났을 때 앱이 강제 종료됨 (크래시)

Unit이란?
// Unit = "반환값 없음"
// Java의 void랑 같아요

suspend fun logout(): Unit
// = logout은 뭔가를 반환하지 않아요
//   그냥 실행만 해요

// 코틀린에서는 반환값 없으면
// 그냥 안 써도 되는데
// Result<Unit> 처럼 쓸 때는 명시해줘야 해요

Result<Unit>
// = 성공/실패는 알려주는데
//   성공했을 때 데이터는 없어요

cf. singleton이랑 inject constructor를 같이 쓰면?

@Singleton  // 앱 전체에서 딱 하나만 만들어
class BookmarkRepository @Inject constructor(
    private val apiService: ApiService,
)

// @Singleton 없으면
// 필요할 때마다 새로 만들어요
// 비효율적이고 데이터가 따로 관리돼요

// @Singleton 있으면
// 처음 한 번만 만들고
// 그 다음엔 같은 거 재사용해요
```
## 5단계 - UiState 정의
화면이 어떤 상태를 가질 수 있어?
```kotlin
// ui/favorites/FavoritesViewModel.kt 상단에 
sealed class FavoritesUiState { 
	object Loading : FavoritesUiState() 
	object Empty : FavoritesUiState() 
	data class Success( 
		val bookmarks: List<Bookmark> 
	) : FavoritesUiState() 
	data class Error(val message: String) : FavoritesUiState() 
}
```
질문
```
// sealed/object/data class/:콜론 ?

sealed class FavoritesUiState {
    object Loading : FavoritesUiState()
    object Empty   : FavoritesUiState()
    data class Success(val bookmarks: List<Bookmark>) : FavoritesUiState()
    data class Error(val message: String) : FavoritesUiState()
}

// sealed = "이 클래스를 상속받을 수 있는 건
//           여기 안에 정의된 것들뿐이야"

// 장점
// when문에서 모든 경우를 강제로 처리해야 해요
when (state) {
    is FavoritesUiState.Loading -> { }
    is FavoritesUiState.Empty   -> { }
    is FavoritesUiState.Success -> { }
    is FavoritesUiState.Error   -> { }
    // 하나라도 빠뜨리면 컴파일 에러!
}

// object
// = 데이터 없이 상태만 표현할 때
object Loading : FavoritesUiState()
// Loading은 그냥 "로딩 중" 이라는 상태만 표현
// 안에 데이터가 필요 없어요

// data class
// = 데이터를 들고 다닐 때
data class Success(val bookmarks: List<Bookmark>)
// Success는 북마크 목록을 들고 있어야 해요
// 그래서 data class

콜론이 뭐야?
// 1. 타입 지정할 때
val name: String = "Q-Guard"
//       ↑ name은 String 타입이야

// 2. 상속/구현할 때
class FavoritesViewModel : ViewModel()
//                        ↑ ViewModel을 상속받아

object Loading : FavoritesUiState()
//             ↑ FavoritesUiState의 자식이야

// 3. 함수 반환 타입
fun getBookmarks(): Result<List<Bookmark>>
//                ↑ 이 함수는 Result<List<Bookmark>>를 반환해
```
## 6. ViewModel 작성
데이터를 가져와서 화면에 전달해줘 = 비즈니스 로직
```kotlin
@HiltViewModel
class FavoritesViewModel @Inject constructor (
	private val bookmarkRepository: BookmarkRepository,
): ViewModel() {
	private val _uiState = MutableStateFlow<FavoritesUiState>(  
		FavoritesUiState.Loading 
) 
val uiState: StateFlow<FavoritesUiState> = _uiState.asStateFlow() 

init { 
	// ViewModel 생성되자마자 데이터 로드 
	loadBookmarks() 
} fun loadBookmarks() { 
	viewModelScope.launch { 
		_uiState.value = FavoritesUiState.Loading 
		bookmarkRepository.getBookmarks().fold( 
			onSuccess = { bookmarks -> 
				_uiState.value = if (bookmarks.isEmpty()) { 
					FavoritesUiState.Empty 
				} else { FavoritesUiState.Success(bookmarks) } 
			}, 
			onFailure = { 
				_uiState.value = FavoritesUiState.Error( 
					it.message ?: "불러오기 실패" 
				) 
			} 
		) 
	} 
}}
```
질문
```
class FavoritesViewModel @Inject constructor(
    private val bookmarkRepository: BookmarkRepository,
) : ViewModel() {
    // 여기 안에 ViewModel의 내용을 적어요
    // 함수, 변수 다 여기에
}

// {} = 클래스의 본문 (몸통)
// 모든 클래스는 {} 안에 내용을 담아요

**변수명 앞에 _ 왜 붙여?**

// 외부에서 직접 수정 못 하게 막으려고
private val _uiState = MutableStateFlow(...)
val uiState = _uiState.asStateFlow()

// _uiState = 내부에서만 수정 가능 (private)
//  uiState = 외부에서 읽기만 가능

// Fragment에서
viewModel.uiState   // ✅ 읽기 가능
viewModel._uiState  // ❌ 접근 불가

자주 쓰는 것들 
.collect { } Flow 데이터 받아오기 
.launch { } 코루틴 시작 
.fold( 성공/실패 처리 
	onSuccess = { }, 
	onFailure = { } 
) 
.asStateFlow() 외부에서 읽기만 가능하게 
.map { } 리스트 변환 

다들 처음엔 공식문서 + 구글링 하면서 배워요 모르는 게 당연해요!

viewmodel을 상속받은게 끝이 아니고 왜 또 {}안에 내가 내용을 추가해야함?
class FavoritesViewModel : ViewModel() { 
	// 여기가 FavoritesViewModel만의 내용 
	// ViewModel 기능 + 내가 추가하는 것들 private val _uiState = ... 
	// 내가 추가한 변수 fun loadBookmarks() { } 
	// 내가 추가한 함수
} // 비유하자면 
// ViewModel = 스마트폰 기본 기능 (전화, 문자) 
// FavoritesViewModel = 내가 앱 설치해서 기능 추가할 수 있는거임
// {} 안에 내용 = 내가 추가하는 기능들
```
## 7. XML 레이아웃 작성
화면이 어떻게 생겼어?
```kotlin
<!-- fragment_favorites.xml -->
<!-- 로딩 / 빈 화면 / 목록 상태에 따라 보여줄 뷰 준비 -->

<ProgressBar android:id="@+id/progress" ... />

<TextView
    android:id="@+id/tv_empty"
    android:text="즐겨찾기가 없어요"
    android:visibility="gone" ... />

<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/rv_bookmarks"
    android:visibility="gone" ... />
```
recyclerview는 목록을 보여줄 때 쓰는 뷰
일반 linear layout으로 목록 만들면 아이템 100개 다 메모리에 올라감
리사이클러뷰는 화면에 보이는것만 메모리에 올림
스크롤하면 안보이는건 재활용 recycle  예) 카카오톡 채팅 목록

---

## 8단계 - Fragment 작성
"UiState 보고 화면 업데이트해줘"
```kotlin
@AndroidEntryPoint
class FavoritesFragment : Fragment() {

    private val viewModel: FavoritesViewModel by viewModels()

    override fun onViewCreated(...) {
        observeState()
    }

    private fun observeState() {
        viewLifecycleOwner.lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is FavoritesUiState.Loading -> {
                            binding.progress.visibility   = View.VISIBLE
                            binding.tvEmpty.visibility    = View.GONE
                            binding.rvBookmarks.visibility = View.GONE
                        }
                        is FavoritesUiState.Empty -> {
                            binding.progress.visibility   = View.GONE
                            binding.tvEmpty.visibility    = View.VISIBLE
                            binding.rvBookmarks.visibility = View.GONE
                        }
                        is FavoritesUiState.Success -> {
                            binding.progress.visibility   = View.GONE
                            binding.tvEmpty.visibility    = View.GONE
                            binding.rvBookmarks.visibility = View.VISIBLE
                            // 어댑터에 데이터 전달
                        }
                        is FavoritesUiState.Error -> {
                            Toast.makeText(
                                requireContext(),
                                state.message,
                                Toast.LENGTH_SHORT
                            ).show()
                        }
                    }
                }
            }
        }
    }
}
```

질문
```
observeState는 꼭 저렇게 해야함?
내가 (또는 팀이) 정한 이름이에요 업계 관례상 observe로 시작하는 경우가 많아요 
observe = "관찰하다" → ViewModel의 상태를 관찰하고 바뀌면 화면 업데이트 
observeUiState, collectState, setupObservers 이런 이름도 많이 써요 정해진 건 없어요

lifecycle이 뭐야?
Fragment의 생명주기예요 Fragment도 태어나고 죽어요 
onCreate → Fragment 생성됨 
onStart → 화면에 보이기 시작 
onResume → 사용자와 상호작용 가능 
onPause → 다른 화면이 위에 올라옴 
onStop → 화면에서 완전히 안 보임 
onDestroy → Fragment 소멸 
왜 중요해? → 화면이 안 보이는데도 데이터 계속 받으면 낭비거든요 repeatOnLifecycle(Lifecycle.State.STARTED) → 화면이 보일 때만 데이터 받고 안 보이면 자동으로 멈춰요
```

---

### 순서 요약 (외워요!)
```
새 기능 만들 때 항상 이 순서

1. 모델      → "이 데이터가 뭘 담아야 해?" 내가 생각해야함(앱에서 편하게 쓸 형식)
2. DTO       → "서버가 어떻게 줘?" api 명세서가 있다면 그걸 보고 만들면됨(서버형식)
3. ApiService → "어디에 요청해?"
4. Repository → "서버 응답을 앱 모델로 변환"
5. UiState   → "화면이 어떤 상태를 가져?"
6. ViewModel  → "데이터 가져와서 상태 업데이트"
7. XML       → "화면이 어떻게 생겼어?"
8. Fragment  → "상태 보고 화면 업데이트"
```

```
cf.
DTO 서버랑 주고받는 그릇 
도메인 모델 앱 안에서 쓰는 그릇 
@어노테이션 특별한 역할 표시 
suspend 기다려도 화면 안 멈춤 
sealed 정해진 상태들만 허용 
object 데이터 없는 상태 
data class 데이터 있는 상태 
_변수명 내부에서만 수정 가능 
Lifecycle Fragment의 생사 
RecyclerView 효율적인 목록 뷰
```