---
title: apidesc
---
---
# Q-Guard FastAPI 백엔드 API 명세서

  

> React 프론트엔드 & Android 앱 호환 API 설계 | 노션 스타일 테이블 형식

  

---

  

## 📋 API 엔드포인트 요약

  

| # | 분류 | 메서드 | 경로 | 설명 | 인증 | 응답시간 |

|---|------|--------|------|------|------|---------|

| 1 | **인증** | `POST` | `/api/auth/register` | 회원가입 | ❌ | < 500ms |

| 2 | | `POST` | `/api/auth/login` | 로그인 | ❌ | < 500ms |

| 3 | | `POST` | `/api/auth/refresh` | 토큰 갱신 | ❌ | < 200ms |

| 4 | **스캔** | `POST` | `/api/v1/scan` | URL 보안 검사 **[핵심]** | ✅ | 0.2-2.0s |

| 5 | | `GET` | `/api/v1/scan/history` | 스캔 히스토리 조회 | ✅ | < 500ms |

| 6 | | `GET` | `/api/v1/scan/stats` | 스캔 통계 조회 | ✅ | < 300ms |

| 7 | **안심URL** | `GET` | `/api/v1/scan/favorites` | 안심 URL 목록 | ✅ | < 200ms |

| 8 | | `POST` | `/api/v1/scan/favorites` | 안심 URL 추가 | ✅ | < 300ms |

| 9 | | `DELETE` | `/api/v1/scan/favorites/{id}` | 안심 URL 삭제 | ✅ | < 200ms |

  

---

  

## 🔐 인증 API

  

### 1️⃣ 회원가입 (Register)

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `POST /api/auth/register` |

| **설명** | 새로운 사용자를 등록합니다 |

| **인증** | ❌ 불필요 |

| **Content-Type** | `application/json` |

  

**요청 파라미터:**

  

| 필드 | 타입 | 필수 | 검증 | 설명 |

|------|------|------|------|------|

| `email` | string | ✓ | 이메일 형식 & 유일 | 사용자 이메일 |

| `password` | string | ✓ | 8자 이상 | 사용자 비밀번호 |

| `username` | string | ✓ | 2-50자 | 표시 이름 |

  

**요청 예시:**

```json

{

"email": "user@example.com",

"password": "SecurePass123!",

"username": "john_doe"

}

```

  

**응답 상태:**

  

| 상태 | 코드 | 설명 |

|------|------|------|

| ✅ 성공 | 201 | 회원가입 완료 |

| ❌ 요청 오류 | 400 | 유효하지 않은 요청 |

| ❌ 충돌 | 409 | 이미 등록된 이메일 |

  

**응답 본문 (201):**

```json

{

"id": 1,

"email": "user@example.com",

"username": "john_doe",

"created_at": "2026-03-15T10:30:00Z"

}

```

  

**에러 응답 (409):**

```json

{

"detail": "Email already registered",

"status": 409

}

```

  

---

  

### 2️⃣ 로그인 (Login)

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `POST /api/auth/login` |

| **설명** | 사용자 인증 후 토큰 발급 |

| **인증** | ❌ 불필요 |

| **Content-Type** | `application/json` |

  

**요청 파라미터:**

  

| 필드 | 타입 | 필수 | 설명 |

|------|------|------|------|

| `email` | string | ✓ | 등록된 이메일 |

| `password` | string | ✓ | 사용자 비밀번호 |

  

**요청 예시:**

```json

{

"email": "user@example.com",

"password": "SecurePass123!"

}

```

  

**응답 상태:**

  

| 상태 | 코드 | 설명 |

|------|------|------|

| ✅ 성공 | 200 | 로그인 완료 |

| ❌ 미인증 | 401 | 잘못된 이메일/비밀번호 |

| ❌ 요청 오류 | 400 | 유효하지 않은 요청 |

  

**응답 본문 (200):**

```json

{

"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",

"refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",

"token_type": "bearer",

"expires_in": 1800

}

```

  

**응답 필드 설명:**

  

| 필드 | 타입 | 설명 |

|------|------|------|

| `access_token` | string | API 호출용 토큰 (30분 유효) |

| `refresh_token` | string | 토큰 갱신용 토큰 (7일 유효) |

| `token_type` | string | 토큰 타입 (항상 "bearer") |

| `expires_in` | integer | 토큰 유효시간 (초 단위) |

  

---

  

### 3️⃣ 토큰 갱신 (Refresh)

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `POST /api/auth/refresh` |

| **설명** | 만료된 Access Token 갱신 |

| **인증** | Refresh Token 사용 |

| **헤더** | `Authorization: Bearer {refresh_token}` |

  

**요청 헤더:**

  

| 헤더 | 값 | 설명 |

|------|---|------|

| `Authorization` | `Bearer {refresh_token}` | 갱신용 토큰 |

  

**응답 상태:**

  

| 상태 | 코드 | 설명 |

|------|------|------|

| ✅ 성공 | 200 | 토큰 갱신 완료 |

| ❌ 미인증 | 401 | 유효하지 않은 토큰 |

  

**응답 본문 (200):**

```json

{

"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",

"token_type": "bearer",

"expires_in": 1800

}

```

  

---

  

## 🔍 스캔 API

  

### 1️⃣ URL 보안 검사 [**핵심**]

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `POST /api/v1/scan` |

| **설명** | URL의 피싱 여부를 ML + WHOIS로 판단 |

| **인증** | ✅ Access Token 필요 |

| **Content-Type** | `application/json` |

| **응답 시간** | 0.2-2.0초 (상세 아래 참조) |

  

**요청 파라미터:**

  

| 필드 | 타입 | 필수 | 검증 | 설명 |

|------|------|------|------|------|

| `url` | string | ✓ | http/https 시작 | 검사할 URL |

  

**요청 예시:**

```json

{

"url": "https://example.com/login"

}

```

  

**응답 상태:**

  

| 상태 | 코드 | 설명 |

|------|------|------|

| ✅ 완료 | 200 | 검사 완료 |

| ❌ 잘못된 요청 | 400 | 잘못된 URL 형식 |

| ❌ 미인증 | 401 | 토큰 없음/만료 |

  

**응답 필드:**

  

| 필드 | 타입 | 설명 | 값 범위 |

|------|------|------|--------|

| `isSafe` | boolean | URL 안전 여부 | true/false |

| `url` | string | 검사한 URL | - |

| `riskScore` | integer | 위험도 점수 | 0-100 |

| `reasons` | array | 위협 판정 사유 | - |

| `isWhitelisted` | boolean | 화이트리스트 등록 여부 | true/false |

| `processedAt` | string | 검사 완료 시간 | ISO 8601 |

  

**응답 시나리오:**

  

**시나리오 1️⃣ - 안전한 URL (200ms):**

```json

{

"isSafe": true,

"url": "https://www.naver.com",

"riskScore": 5,

"reasons": [],

"isWhitelisted": false,

"processedAt": "2026-03-15T10:35:22Z"

}

```

  

**시나리오 2️⃣ - 위험한 URL (1-2초):**

```json

{

"isSafe": false,

"url": "http://phishing-site.com/auth",

"riskScore": 94,

"reasons": [

"도메인 생성일 3일 미만",

"HTTPS 미사용",

"높은 피싱 특성 감지 (ML Score: 92%)"

],

"isWhitelisted": false,

"processedAt": "2026-03-15T10:35:22Z"

}

```

  

**시나리오 3️⃣ - 화이트리스트 (즉시 응답 <200ms):**

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

  

---

  

### 2️⃣ 스캔 히스토리 조회

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `GET /api/v1/scan/history` |

| **설명** | 사용자의 스캔 기록 조회 (페이지네이션) |

| **인증** | ✅ Access Token 필요 |

| **응답 시간** | < 500ms |

  

**쿼리 파라미터:**

  

| 파라미터 | 타입 | 기본값 | 범위 | 설명 |

|---------|------|-------|------|------|

| `limit` | integer | 20 | 1-100 | 반환할 항목 수 |

| `offset` | integer | 0 | ≥0 | 시작 위치 (페이지네이션) |

  

**요청 예시:**

```

GET /api/v1/scan/history?limit=20&offset=0

Authorization: Bearer {access_token}

```

  

**응답 상태:**

  

| 상태 | 코드 | 설명 |

|------|------|------|

| ✅ 성공 | 200 | 조회 완료 |

| ❌ 미인증 | 401 | 토큰 없음/만료 |

  

**응답 필드:**

  

| 필드 | 타입 | 설명 |

|------|------|------|

| `total` | integer | 전체 스캔 기록 수 |

| `limit` | integer | 요청한 limit 값 |

| `offset` | integer | 요청한 offset 값 |

| `history` | array | 스캔 기록 배열 |

| `history[].id` | integer | 스캔 결과 ID |

| `history[].url` | string | 검사한 URL |

| `history[].isSafe` | boolean | 안전 여부 |

| `history[].riskScore` | integer | 위험도 점수 |

| `history[].reasons` | array | 위협 사유 |

| `history[].scannedAt` | string | 스캔 시간 (ISO 8601) |

  

**응답 예시 (200):**

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

  

### 3️⃣ 스캔 통계 조회

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `GET /api/v1/scan/stats` |

| **설명** | 사용자의 스캔 통계 조회 |

| **인증** | ✅ Access Token 필요 |

| **응답 시간** | < 300ms |

  

**요청 예시:**

```

GET /api/v1/scan/stats

Authorization: Bearer {access_token}

```

  

**응답 필드:**

  

| 필드 | 타입 | 설명 | 관계식 |

|------|------|------|-------|

| `totalScans` | integer | 전체 스캔 수 | - |

| `safeCount` | integer | 안전 URL 개수 | - |

| `maliciousCount` | integer | 위험 URL 개수 | `totalScans - safeCount` |

| `safePercentage` | number | 안전 비율 (%) | `(safeCount/totalScans) * 100` |

| `maliciousPercentage` | number | 위험 비율 (%) | `100 - safePercentage` |

| `thisWeek` | integer | 이번 주 스캔 수 | - |

| `thisMonth` | integer | 이번 달 스캔 수 | - |

  

**응답 예시 (200):**

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

  

## ⭐ 안심 URL (화이트리스트) API

  

### 1️⃣ 안심 URL 목록 조회

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `GET /api/v1/scan/favorites` |

| **설명** | 사용자의 안심 URL 목록 조회 |

| **인증** | ✅ Access Token 필요 |

| **응답 시간** | < 200ms |

  

**요청 예시:**

```

GET /api/v1/scan/favorites

Authorization: Bearer {access_token}

```

  

**응답 필드:**

  

| 필드 | 타입 | 설명 |

|------|------|------|

| `total` | integer | 전체 안심 URL 개수 |

| `favorites` | array | 안심 URL 배열 |

| `favorites[].id` | integer | 안심 URL ID |

| `favorites[].url` | string | 안심 등록된 URL |

| `favorites[].addedAt` | string | 등록 시간 (ISO 8601) |

  

**응답 예시 (200):**

```json

{

"total": 2,

"favorites": [

{

"id": 1,

"url": "https://www.starbucks.co.kr",

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

  

### 2️⃣ 안심 URL 추가

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `POST /api/v1/scan/favorites` |

| **설명** | URL을 안심 목록에 추가 |

| **인증** | ✅ Access Token 필요 |

| **Content-Type** | `application/json` |

| **응답 시간** | < 300ms |

  

**요청 파라미터:**

  

| 필드 | 타입 | 필수 | 검증 | 설명 |

|------|------|------|------|------|

| `url` | string | ✓ | http/https 시작 | 등록할 URL |

  

**요청 예시:**

```json

{

"url": "https://www.google.com"

}

```

  

**응답 상태:**

  

| 상태 | 코드 | 설명 |

|------|------|------|

| ✅ 생성됨 | 201 | 안심 URL 등록 완료 |

| ❌ 요청 오류 | 400 | 유효하지 않은 URL |

| ❌ 미인증 | 401 | 토큰 없음/만료 |

| ❌ 충돌 | 409 | 이미 등록된 URL |

  

**응답 본문 (201):**

```json

{

"id": 3,

"url": "https://www.google.com",

"addedAt": "2026-03-15T10:40:00Z",

"message": "URL added to favorites"

}

```

  

**에러 응답 (409):**

```json

{

"detail": "URL already in favorites",

"status": 409

}

```

  

---

  

### 3️⃣ 안심 URL 삭제

  

| 항목 | 내용 |

|------|------|

| **엔드포인트** | `DELETE /api/v1/scan/favorites/{id}` |

| **설명** | 안심 URL 목록에서 제거 |

| **인증** | ✅ Access Token 필요 |

| **응답 시간** | < 200ms |

  

**경로 파라미터:**

  

| 파라미터 | 타입 | 필수 | 설명 |

|---------|------|------|------|

| `id` | integer | ✓ | 삭제할 안심 URL ID |

  

**요청 예시:**

```

DELETE /api/v1/scan/favorites/3

Authorization: Bearer {access_token}

```

  

**응답 상태:**

  

| 상태 | 코드 | 설명 |

|------|------|------|

| ✅ 삭제됨 | 204 | 삭제 성공 |

| ❌ 미인증 | 401 | 토큰 없음/만료 |

| ❌ 찾을 수 없음 | 404 | 해당 안심 URL 없음 |

  

**응답 본문 (204):**

```

(응답 바디 없음)

```

  

---

  

## 📊 HTTP 상태 코드

  

| 코드 | 상태명 | 설명 |

|------|--------|------|

| **200** | OK | 요청 성공 |

| **201** | Created | 리소스 생성 성공 |

| **204** | No Content | 요청 성공 (응답 없음) |

| **400** | Bad Request | 잘못된 요청 형식 |

| **401** | Unauthorized | 인증 필요 (토큰 없음/만료) |

| **403** | Forbidden | 권한 없음 |

| **404** | Not Found | 리소스를 찾을 수 없음 |

| **409** | Conflict | 리소스 충돌 (중복 등) |

| **500** | Internal Server Error | 서버 오류 |

  

---

  

## 🔗 공통 요청 헤더

  

| 헤더 | 필수 | 값 | 설명 |

|------|------|---|------|

| `Content-Type` | ✓ | `application/json` | 요청 본문 형식 |

| `Authorization` | 조건부 | `Bearer {access_token}` | JWT 인증 토큰 (특정 엔드포인트) |

| `User-Agent` | ❌ | `Q-Guard/1.0 (Android)` | 클라이언트 정보 |

  

---

  

## ⏱️ 응답 시간 가이드

  

| 시나리오 | 응답 시간 | 설명 | 캐시 여부 |

|--------|---------|------|---------|

| 💚 **화이트리스트 URL** | **< 200ms** | DB 조회만 진행 (즉시 반환) | ❌ DB |

| ⚡ **캐시된 URL** | **< 500ms** | Redis에서 조회 | ✅ Redis |

| 🤖 **새로운 URL** | **1.0~2.0초** | ML 분석 + WHOIS 조회 필요 | ❌ 신규 |

  

---

  

## 📝 에러 응답 포맷

  

모든 에러는 다음 형식으로 반환:

  

```json

{

"detail": "에러 설명",

"status": 400,

"timestamp": "2026-03-15T10:35:22Z"

}

```

  

| 필드 | 타입 | 설명 |

|------|------|------|

| `detail` | string | 에러 메시지 |

| `status` | integer | HTTP 상태 코드 |

| `timestamp` | string | 에러 발생 시간 (ISO 8601) |

  

---

  

## 🚀 클라이언트 통합 예시

  

### JavaScript (React)

```javascript

const apiClient = {

baseURL: 'https://api.qguard.com',

async scanUrl(url, accessToken) {

const response = await fetch(`${this.baseURL}/api/v1/scan`, {

method: 'POST',

headers: {

'Content-Type': 'application/json',

'Authorization': `Bearer ${accessToken}`

},

body: JSON.stringify({ url })

});

return await response.json();

},

async getHistory(accessToken, limit = 20, offset = 0) {

const query = new URLSearchParams({ limit, offset });

const response = await fetch(

`${this.baseURL}/api/v1/scan/history?${query}`,

{

headers: { 'Authorization': `Bearer ${accessToken}` }

}

);

return await response.json();

}

};

```

  

### Kotlin (Android)

```kotlin

interface QGuardApi {

@POST("api/v1/scan")

suspend fun scanUrl(

@Body request: ScanRequest

): ScanResponse

  

@GET("api/v1/scan/history")

suspend fun getHistory(

@Query("limit") limit: Int = 20,

@Query("offset") offset: Int = 0

): HistoryResponse

}

  

// Retrofit 클라이언트

val retrofit = Retrofit.Builder()

.baseUrl("https://api.qguard.com/")

.addConverterFactory(GsonConverterFactory.create())

.client(httpClient)

.build()

  

val apiService = retrofit.create(QGuardApi::class.java)

```