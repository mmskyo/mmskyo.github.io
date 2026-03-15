---
title: apidesc
---
---
# Q-Guard FastAPI 백엔드 API 명세서

  

> React 프론트엔드 & Android 앱 호환 API 설계

  

## 목차

1. [인증 API](#인증-api)

2. [스캔 API](#스캔-api)

3. [안심-url-api](#안심-url-api)

4. [응답 코드](#응답-코드)

  

---

  

## 인증 API

  

### 1. 회원가입

```

POST /api/auth/register

```

  

**요청 (Request):**

```json

{

"email": "user@example.com",

"password": "SecurePass123!",

"username": "john_doe"

}

```

  

**응답 200 (성공):**

```json

{

"id": 1,

"email": "user@example.com",

"username": "john_doe",

"created_at": "2026-03-15T10:30:00Z"

}

```

  

**응답 400 (이메일 중복):**

```json

{

"detail": "Email already registered"

}

```

  

---

  

### 2. 로그인

```

POST /api/auth/login

```

  

**요청 (Request):**

```json

{

"email": "user@example.com",

"password": "SecurePass123!"

}

```

  

**응답 200 (성공):**

```json

{

"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",

"refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",

"token_type": "bearer",

"expires_in": 3600

}

```

  

**응답 401 (인증 실패):**

```json

{

"detail": "Invalid email or password"

}

```

  

---

  

### 3. 토큰 갱신

```

POST /api/auth/refresh

Authorization: Bearer {refresh_token}

```

  

**응답 200:**

```json

{

"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",

"token_type": "bearer",

"expires_in": 3600

}

```

  

---

  

## 스캔 API

  

### 1. URL 보안 검사 (핵심 기능)

```

POST /api/v1/scan

Authorization: Bearer {access_token}

Content-Type: application/json

```

  

**요청 (Request):**

```json

{

"url": "https://example.com/login"

}

```

  

**응답 200 (안전한 URL):**

```json

{

"isSafe": true,

"url": "https://example.com/login",

"riskScore": 8,

"reasons": [],

"isWhitelisted": false,

"processedAt": "2026-03-15T10:35:22Z"

}

```

  

**응답 200 (위험한 URL):**

```json

{

"isSafe": false,

"url": "http://phishing-attempt-login.com/auth",

"riskScore": 94,

"reasons": [

"도메인 생성일 7일 미만 (단기 폐쇄 목적 의심)",

"HTTPS 미사용 또는 인증서 오류",

"난독화된 경로 포함"

],

"isWhitelisted": false,

"processedAt": "2026-03-15T10:35:22Z"

}

```

  

**응답 200 (화이트리스트 URL - 즉시 응답):**

```json

{

"isSafe": true,

"url": "https://www.starbucks.co.kr",

"riskScore": 0,

"reasons": [],

"isWhitelisted": true,

"processedAt": "2026-03-15T10:35:22Z"

}

```

  

**응답 400 (잘못된 URL):**

```json

{

"detail": "Invalid URL format. Must include http:// or https://"

}

```

  

---

  

### 2. 스캔 히스토리 조회

```

GET /api/v1/scan/history?limit=20&offset=0

Authorization: Bearer {access_token}

```

  

**응답 200:**

```json

{

"total": 45,

"limit": 20,

"offset": 0,

"history": [

{

"id": 1,

"url": "https://malware-site.com",

"isSafe": false,

"riskScore": 96,

"reasons": ["알려진 악성 도메인"],

"scannedAt": "2026-03-15T14:30:00Z"

},

{

"id": 2,

"url": "https://m.naver.com",

"isSafe": true,

"riskScore": 5,

"reasons": [],

"scannedAt": "2026-03-15T09:15:00Z"

}

]

}

```

  

---

  

### 3. 스캔 통계

```

GET /api/v1/scan/stats

Authorization: Bearer {access_token}

```

  

**응답 200:**

```json

{

"totalScans": 150,

"safeCount": 130,

"maliciousCount": 20,

"safePercentage": 86.67,

"maliciousPercentage": 13.33,

"thisWeek": 45,

"thisMonth": 120

}

```

  

---

  

## 안심-URL API

  

### 1. 안심 URL 목록 조회

```

GET /api/v1/scan/favorites

Authorization: Bearer {access_token}

```

  

**응답 200:**

```json

{

"total": 2,

"favorites": [

{

"id": 1,

"url": "https://www.starbucks.co.kr/menu/drink_view.do",

"addedAt": "2026-03-10T15:20:00Z"

},

{

"id": 2,

"url": "https://m.naver.com",

"addedAt": "2026-03-05T10:00:00Z"

}

]

}

```

  

---

  

### 2. 안심 URL 추가 (즐겨찾기)

```

POST /api/v1/scan/favorites

Authorization: Bearer {access_token}

Content-Type: application/json

```

  

**요청:**

```json

{

"url": "https://www.google.com"

}

```

  

**응답 201 (생성됨):**

```json

{

"id": 3,

"url": "https://www.google.com",

"addedAt": "2026-03-15T10:40:00Z",

"message": "URL added to favorites"

}

```

  

**응답 409 (이미 등록됨):**

```json

{

"detail": "URL already in favorites"

}

```

  

---

  

### 3. 안심 URL 제거

```

DELETE /api/v1/scan/favorites/{url_id}

Authorization: Bearer {access_token}

```

  

**응답 204 (삭제됨):**

```

(응답 바디 없음)

```

  

**응답 404 (찾을 수 없음):**

```json

{

"detail": "Favorite not found"

}

```

  

---

  

## 응답 코드

  

| 상태 코드 | 의미 | 설명 |

|---------|------|------|

| **200** | OK | 요청 성공 |

| **201** | Created | 리소스 생성 성공 |

| **204** | No Content | 삭제 성공 |

| **400** | Bad Request | 잘못된 요청 (URL 형식 오류 등) |

| **401** | Unauthorized | 인증 토큰 없음 또는 만료 |

| **403** | Forbidden | 권한 없음 |

| **409** | Conflict | 중복 리소스 (이미 가입된 이메일 등) |

| **500** | Internal Server Error | 서버 오류 |

  

---

  

## 공통 요청 헤더

  

```

Authorization: Bearer {access_token}

Content-Type: application/json

User-Agent: Q-Guard/1.0 (Android; iOS)

```

  

---

  

## React/Android 클라이언트 사용 예시

  

### JavaScript (React)

```javascript

const checkUrlWithFastAPI = async (url) => {

const token = localStorage.getItem('access_token');

try {

const response = await fetch('https://api.qguard.com/api/v1/scan', {

method: 'POST',

headers: {

'Authorization': `Bearer ${token}`,

'Content-Type': 'application/json'

},

body: JSON.stringify({ url })

});

if (response.ok) {

return await response.json();

// { isSafe, url, riskScore, reasons, isWhitelisted }

}

} catch (error) {

console.error('Scan failed:', error);

}

};

```

  

### Kotlin (Android)

```kotlin

// Retrofit 인터페이스

interface QGuardApi {

@POST("api/v1/scan")

suspend fun scanUrl(

@Header("Authorization") token: String,

@Body request: ScanRequest

): ScanResponse

@GET("api/v1/scan/history")

suspend fun getHistory(

@Header("Authorization") token: String,

@Query("limit") limit: Int = 20

): HistoryResponse

}

  

// 사용

val result = apiService.scanUrl("Bearer $token", ScanRequest(url))

```

  

---

  

## 성능 목표

  

| 시나리오 | 응답 시간 | 설명 |

|--------|---------|------|

| 화이트리스트 URL | **< 200ms** | DB 조회만 (즉시 반환) |

| 캐시된 URL | **< 500ms** | Redis에서 조회 |

| 새로운 URL | **1.0~2.0초** | ML 분석 + WHOIS 조회 |

  

---

  

## 에러 핸들링

  

모든 에러는 다음 형식으로 응답됩니다:

  

```json

{

"detail": "설명",

"status": 400,

"timestamp": "2026-03-15T10:35:22Z"

}

```