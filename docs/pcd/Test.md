---
title: test
---
---
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


# android Test

```kotlin
package com.qguarder.android.ui.favorites  
  
import androidx.test.espresso.Espresso.onView  
import androidx.test.espresso.assertion.ViewAssertions.matches  
import androidx.test.espresso.matcher.ViewMatchers.isDisplayed  
import androidx.test.espresso.matcher.ViewMatchers.withId  
import androidx.test.espresso.matcher.ViewMatchers.withText  
import com.qguarder.android.R  
import com.qguarder.android.data.local.FavoriteCacheEntity  
import com.qguarder.android.launchFragmentInHiltContainer  
import dagger.hilt.android.testing.HiltAndroidRule  
import dagger.hilt.android.testing.HiltAndroidTest  
import org.hamcrest.CoreMatchers.not  
import org.junit.Before  
import org.junit.Rule  
import org.junit.Test  
  
@HiltAndroidTest  
class FavoritesFragmentTest {  
  
    @get:Rule  
    var hiltRule = HiltAndroidRule(this)  
  
    @Before  
    fun setUp() {  
        hiltRule.inject()  
    }  
  
    @Test  
    fun test_favorite_empty() {  
        // [Given] Hilt 전용 컨테이너를 사용하여 Fragment 실행  
        launchFragmentInHiltContainer<FavoritesFragment>()  
  
        // [Then] UI 상태 확인  
        // 1. "즐겨찾기가 없습니다" 텍스트가 보이는지 확인  
        onView(withText("즐겨찾기가 없습니다"))  
            .check(matches(isDisplayed()))  
  
        // 2. 비어있음 레이아웃(layout_empty)이 보이는지 확인  
        onView(withId(R.id.layout_empty))  
            .check(matches(isDisplayed()))  
  
        // 3. 하단 안내 문구가 보이는지 확인  
        onView(withText("스캔 결과 화면에서 즐겨찾기를 추가해보세요"))  
            .check(matches(isDisplayed()))  
  
        // 4. 목록(RecyclerView)이 화면에서 안 보이는지(GONE) 확인  
        onView(withId(R.id.rv_favorites))  
            .check(matches(not(isDisplayed())))  
    }  
  
    @Test  
    fun test_favorite_not_empty() {  
        // 더미 목록  
        val fakeFavorites = listOf(  
            FavoriteCacheEntity(  
                favoriteId = "1",  
                url = "https://www.naver.com",  
                title = "네이버",  
                riskLevel = "SECURED",  
                addedAt = "2024-03-20"  
            ),  
            FavoriteCacheEntity(  
                favoriteId = "2",  
                url = "https://www.google.com",  
                title = "구글",  
                riskLevel = "SECURED",  
                addedAt = "2024-03-21"  
            )  
        )  
  
        launchFragmentInHiltContainer<FavoritesFragment>()  
  
        // 1. 목록(RecyclerView)이 이제 보여야 함  
        onView(withId(R.id.rv_favorites))  
            .check(matches(isDisplayed()))  
  
        // 2. 비어있음 안내 레이아웃은 숨겨져야 함  
        onView(withId(R.id.layout_empty))  
            .check(matches(not(isDisplayed())))  
  
        // 3. 내가 넣은 "네이버"라는 글자가 화면에 진짜 있는지 확인  
        onView(withText("네이버"))  
            .check(matches(isDisplayed()))  
    }  
}
```

테스트 에러 not empty 실패
```
androidx.test.espresso.base.AssertionErrorHandler$AssertionFailedWithCauseError: '(view has effective visibility <VISIBLE> and view.getGlobalVisibleRect() to return non-empty rectangle)' doesn't match the selected view.
Expected: (view has effective visibility <VISIBLE> and view.getGlobalVisibleRect() to return non-empty rectangle)
Got: view.getVisibility() was <GONE>
View Details: RecyclerView{id=2131231165, res-name=rv_favorites, visibility=GONE, width=0, height=0, has-focus=false, has-focusable=false, has-window-focus=true, is-clickable=false, is-enabled=true, is-focused=false, is-focusable=true, is-layout-requested=true, is-selected=false, layout-params=androidx.coordinatorlayout.widget.CoordinatorLayout$LayoutParams@YYYYYY, tag=null, root-is-layout-requested=false, has-input-connection=false, x=0.0, y=168.0, child-count=0}
```
