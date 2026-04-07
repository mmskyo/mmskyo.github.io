---
title: server
parent: pcd
---
---
즐겨찾기가 **내 폰 로컬 DB에만** 저장됨. 서버랑 아직 안 연결됨.

  

**내일 해야 할 것 (순서대로)**

  

**STEP 1 — Postman으로 API 먼저 살아있는지 확인 (5분)**

코드 건드리기 전에 서버 API가 정상인지 먼저 확인합니다.

**1-1. 로그인해서 토큰 받기**

POST {{BASE_URL}}/api/v1/auth/login

Body (JSON):

{

  "email": "테스트계정@email.com",

  "password": "비밀번호"

}

응답에서 access_token 복사해두기

**1-2. 북마크 목록 조회**

GET {{BASE_URL}}/api/v1/users/me/bookmarks

Headers:

  Authorization: Bearer {위에서 복사한 토큰}

→ 200 응답 오면 OK

**1-3. 북마크 추가**

POST {{BASE_URL}}/api/v1/users/me/bookmarks

Headers:

  Authorization: Bearer {토큰}

Body (JSON):

{

  "url": "https://www.naver.com",

  "title": "네이버"

}

→ 응답에서 bookmark_id 복사해두기

**1-4. 북마크 삭제**

DELETE {{BASE_URL}}/api/v1/users/me/bookmarks/{위에서 받은 bookmark_id}

Headers:

  Authorization: Bearer {토큰}

→ 여기까지 다 되면 서버 OK, 코드 작업 시작

  

**STEP 2 — BookmarkRepository 파일 만들기**

경로: app/src/main/java/com/qguarder/android/data/remote/BookmarkRepository.kt

이 파일이 서버 API 호출을 담당합니다. 새 파일로 만드세요.

package com.qguarder.android.data.remote

  

import com.qguarder.android.data.local.FavoriteCacheDao

import com.qguarder.android.data.local.FavoriteCacheEntity

import com.qguarder.android.data.remote.dto.AddBookmarkRequest

import javax.inject.Inject

import javax.inject.Singleton

  

@Singleton

class BookmarkRepository @Inject constructor(

    private val apiService: ApiService,

    private val favoriteCacheDao: FavoriteCacheDao

) {

    // 서버에서 북마크 목록 가져와서 로컬 DB에 저장

    suspend fun syncFromServer() {

        val response = apiService.getBookmarks()

        if (response.isSuccessful) {

            val items = response.body()?.items ?: return

            val entities = items.map {

                FavoriteCacheEntity(

                    favoriteId = it.bookmarkId,

                    url        = it.url,

                    title      = it.title,

                    riskLevel  = it.riskLevel,

                    addedAt    = it.createdAt

                )

            }

            favoriteCacheDao.clearAll()

            favoriteCacheDao.insertAll(entities)

        }

    }

  

    // 서버에 북마크 추가 → 서버가 준 bookmark_id로 로컬 DB에 저장

    suspend fun add(url: String, title: String?, riskLevel: String): Boolean {

        val response = apiService.addBookmark(AddBookmarkRequest(url, title))

        if (response.isSuccessful) {

            val body = response.body() ?: return false

            favoriteCacheDao.insert(

                FavoriteCacheEntity(

                    favoriteId = body.bookmarkId,  // 서버가 준 ID 사용

                    url        = url,

                    title      = title,

                    riskLevel  = riskLevel,

                    addedAt    = body.createdAt

                )

            )

            return true

        }

        return false

    }

  

    // 서버에서 북마크 삭제 → 로컬 DB에서도 삭제

    suspend fun remove(url: String) {

        val entity = favoriteCacheDao.getByUrl(url) ?: return

        val response = apiService.deleteBookmark(entity.favoriteId)

        if (response.isSuccessful) {

            favoriteCacheDao.deleteByUrl(url)

        }

    }

}

  

**STEP 3 — FavoritesViewModel에 서버 sync 연결**

FavoritesViewModel.kt 열기

상단 @Inject constructor에 BookmarkRepository 추가하고, init 블록에서 서버 sync 호출:

@HiltViewModel

class FavoritesViewModel @Inject constructor(

    private val favoriteRepository: FavoriteRepository,

    private val bookmarkRepository: BookmarkRepository  // 추가

) : ViewModel() {

  

    val favorites: StateFlow<List<FavoriteCacheEntity>> =

        favoriteRepository.getAll()

            .stateIn(

                scope = viewModelScope,

                started = SharingStarted.WhileSubscribed(5000),

                initialValue = emptyList()

            )

  

    init {

        // 화면 열릴 때 서버에서 최신 목록 가져오기

        viewModelScope.launch {

            bookmarkRepository.syncFromServer()

        }

    }

  

    fun delete(url: String) {

        viewModelScope.launch {

            bookmarkRepository.remove(url)  // 로컬만→서버+로컬로 변경

        }

    }

}

  

**STEP 4 — ResultViewModel에 서버 연동**

ResultViewModel.kt 열기

생성자에 BookmarkRepository 추가하고 toggleFavorite() 수정:

@HiltViewModel

class ResultViewModel @Inject constructor(

    private val favoriteRepository: FavoriteRepository,

    private val bookmarkRepository: BookmarkRepository  // 추가

) : ViewModel() {

  

    private val _isFavorite = MutableStateFlow(false)

    val isFavorite = _isFavorite.asStateFlow()

  

    fun checkFavorite(url: String) {

        viewModelScope.launch {

            _isFavorite.value = favoriteRepository.isFavorite(url)

        }

    }

  

    fun toggleFavorite(url: String, title: String?, riskLevel: String) {

        viewModelScope.launch {

            if (_isFavorite.value) {

                bookmarkRepository.remove(url)      // 서버+로컬 삭제

            } else {

                bookmarkRepository.add(url, title, riskLevel)  // 서버+로컬 추가

            }

            _isFavorite.value = !_isFavorite.value

        }

    }

}

  

**STEP 5 — 에뮬레이터로 실제 확인**

순서대로 해보기:

1. 앱 실행 → 로그인
2. 즐겨찾기 탭 열기 → Logcat에서 에러 없는지 확인
3. Postman에서 직접 북마크 1개 추가 → 앱 즐겨찾기 탭 다시 열어서 뜨는지 확인
4. 앱에서 스캔 결과 화면 → 즐겨찾기 버튼 눌러서 추가 → Postman GET으로 서버에 저장됐는지 확인
5. 앱에서 삭제 → Postman GET으로 서버에서도 없어졌는지 확인

  

**자주 나오는 에러 & 해결법**

|   |   |   |
|---|---|---|
|**에러**|**원인**|**해결**|
|401 Unauthorized|토큰 만료 또는 로그인 안 됨|앱에서 로그아웃 후 재로그인|
|NullPointerException on body|서버 응답 body가 null|response.body() ?: return 체크|
|목록이 안 업데이트됨|syncFromServer 안 불림|init 블록 있는지 확인|
|삭제가 서버에 안 됨|favoriteId가 서버 ID랑 다름|Step 2의 add()에서 서버 bookmarkId 쓰는지 확인|