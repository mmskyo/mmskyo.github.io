---
title: roadmap
---
---
# 백엔드 + Android 개발 계획서
> 3주차 시작 기준 / 전체 15주 중 남은 12주 로드맵

---

## 역할 한 줄 정리

| 역할 | 담당 |
|------|------|
| 클라우드팀 (서버) | AWS Lambda, RDS, Redis, S3, ML 추론, MLOps |
| 내 역할 (백엔드 + Android) | API 엔드포인트 구현 + Android UI 연동 |

> 백엔드는 "클라우드팀 서버 결과를 Android가 이해할 수 있는 JSON으로 포장하는 번역기"입니다.
> Android는 "그 JSON을 받아서 화면에 표시하는 앱"입니다.

---

## 전체 일정 요약

| 주차 | 백엔드 | Android |
|------|--------|---------|
| 3주차 | 환경 세팅 + Auth API | 프로젝트 세팅 + 로그인 화면 |
| 4주차 | Scan API 스텁 | 카메라 스캔 화면 |
| 5주차 | Scan API 완성 | 결과 화면 (점수 + XAI) |
| 6주차 | 방문기록 / 즐겨찾기 API | 방문기록 / 즐겨찾기 화면 |
| 7주차 | 신고 API + 블랙리스트 | 신고 버튼 + 블랙리스트 화면 |
| 8주차 | 로그 관리 API (캘린더) | 캘린더 화면 |
| 9주차 | 통합 테스트 + 에러 처리 | 통합 테스트 + 에러 처리 |
| 10주차 | 성능 최적화 + 보안 점검 | UI 다듬기 |
| 11~13주차 | 버그 수정 + 클라우드팀 연동 | 버그 수정 + 클라우드팀 연동 |
| 14~15주차 | 최종 발표 준비 | 최종 발표 준비 |

---

---

# 3주차 — 환경 세팅 + Auth

## 백엔드

### 목표
- FastAPI 프로젝트 구조 잡기
- 회원가입 / 로그인 / 토큰 갱신 / 로그아웃 API 완성

---

### 원리 설명: FastAPI가 뭔가요?

Python으로 API 서버를 만드는 프레임워크예요.
Android가 "이 URL 분석해줘"라고 요청을 보내면,
FastAPI가 그 요청을 받아서 처리하고 JSON으로 돌려줘요.

```
Android → (HTTP 요청) → FastAPI → (JSON 응답) → Android
```

Flask랑 비슷하지만 두 가지가 다르게 좋아요.
1. async/await 지원 → 요청 여러 개를 동시에 처리 가능
2. Pydantic 자동 연동 → JSON 형식이 틀리면 자동으로 에러 잡아줌

---

### 원리 설명: JWT가 뭔가요?

로그인하면 서버가 "너 맞아" 라는 증명서를 줘요. 그게 JWT(토큰)예요.
이후 모든 요청에 이 토큰을 같이 보내면 서버가 신원을 확인해요.

```
1. 로그인 → 서버가 access_token 발급
2. 이후 요청마다 → Header에 "Bearer {토큰}" 붙여서 전송
3. 서버 → 토큰 검증 → 맞으면 처리, 틀리면 401 에러
```

access_token은 수명이 짧아요 (30분).
수명이 긴 refresh_token(7일)으로 access_token을 재발급받아요.
refresh_token은 HttpOnly 쿠키에 저장 → JavaScript로 접근 불가 → 보안상 안전.

---

### 튜토리얼: 프로젝트 세팅

```bash
# 1. 폴더 생성
mkdir phishing-backend && cd phishing-backend

# 2. 가상환경 생성 (패키지를 프로젝트별로 격리)
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. 패키지 설치
pip install fastapi==0.111.0 uvicorn==0.30.0 pydantic==2.7.0 \
            python-jose==3.3.0 passlib==1.7.4 bcrypt==4.0.1 \
            sqlalchemy==2.0.30 asyncpg==0.29.0 alembic==1.13.1 \
            python-dotenv==1.0.1

# 4. requirements.txt 저장
pip freeze > requirements.txt
```

폴더 구조 만들기:

```
phishing-backend/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py       ← 환경변수 관리
│   │   ├── security.py     ← JWT 발급/검증
│   │   └── dependencies.py ← 공통 의존성 (DB연결, 현재유저 등)
│   ├── api/
│   │   └── v1/
│   │       ├── __init__.py
│   │       └── auth.py     ← 회원가입/로그인 엔드포인트
│   ├── models/
│   │   ├── __init__.py
│   │   └── db.py           ← DB 테이블 정의
│   └── schemas/
│       ├── __init__.py
│       └── auth.py         ← 요청/응답 JSON 형식 정의
├── alembic/
├── .env
└── requirements.txt
```

---

### 튜토리얼: 환경변수 설정

`.env` 파일 (절대 Git에 올리지 마세요):

```
DATABASE_URL=postgresql+asyncpg://user:password@localhost:5432/phishing_db
JWT_SECRET_KEY=여기에_아무_랜덤_문자열_넣기_예시_abc123xyz
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7
```

`app/core/config.py`:

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str
    JWT_SECRET_KEY: str
    JWT_ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    REFRESH_TOKEN_EXPIRE_DAYS: int = 7

    class Config:
        env_file = ".env"

settings = Settings()
```

---

### 튜토리얼: JWT 발급/검증

`app/core/security.py`:

```python
from datetime import datetime, timedelta
from jose import jwt, JWTError
from passlib.context import CryptContext
from app.core.config import settings

# bcrypt로 비밀번호 해시
pwd_context = CryptContext(schemes=["bcrypt"])

def hash_password(password: str) -> str:
    """비밀번호를 해시값으로 변환 (DB에 이걸 저장)"""
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    """입력한 비밀번호와 DB의 해시값이 일치하는지 확인"""
    return pwd_context.verify(plain, hashed)

def create_access_token(user_id: str) -> str:
    """access_token 발급 (수명 30분)"""
    expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    payload = {"sub": user_id, "exp": expire, "type": "access"}
    return jwt.encode(payload, settings.JWT_SECRET_KEY, algorithm=settings.JWT_ALGORITHM)

def create_refresh_token(user_id: str) -> str:
    """refresh_token 발급 (수명 7일)"""
    expire = datetime.utcnow() + timedelta(days=settings.REFRESH_TOKEN_EXPIRE_DAYS)
    payload = {"sub": user_id, "exp": expire, "type": "refresh"}
    return jwt.encode(payload, settings.JWT_SECRET_KEY, algorithm=settings.JWT_ALGORITHM)

def decode_token(token: str) -> dict:
    """토큰 해독 — 만료됐거나 위조됐으면 에러"""
    try:
        return jwt.decode(token, settings.JWT_SECRET_KEY, algorithms=[settings.JWT_ALGORITHM])
    except JWTError:
        return None
```

---

### 튜토리얼: 응답 스키마 정의

`app/schemas/auth.py`:

```python
from pydantic import BaseModel, EmailStr
from datetime import datetime

# 회원가입 요청 형식
class RegisterRequest(BaseModel):
    email: EmailStr          # 이메일 형식 자동 검증
    password: str
    nickname: str

# 로그인 요청 형식
class LoginRequest(BaseModel):
    email: EmailStr
    password: str

# 토큰 응답 형식
class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"

# 회원가입 응답 형식
class RegisterResponse(BaseModel):
    user_id: str
    email: str
    nickname: str
    created_at: datetime
```

> Pydantic이 하는 일: Android가 email 필드에 "abc"를 보내면
> EmailStr이 자동으로 "올바른 이메일 형식이 아닙니다" 에러를 잡아줘요.
> 직접 검증 코드를 짤 필요가 없어요.

---

### 튜토리얼: Auth 엔드포인트

`app/api/v1/auth.py`:

```python
from fastapi import APIRouter, HTTPException, Response, Cookie
from app.schemas.auth import RegisterRequest, LoginRequest, TokenResponse, RegisterResponse
from app.core.security import hash_password, verify_password, create_access_token, create_refresh_token, decode_token

router = APIRouter(prefix="/api/v1/auth", tags=["auth"])

# 임시 유저 저장소 (나중에 DB로 교체)
fake_users = {}

@router.post("/register", response_model=RegisterResponse, status_code=201)
async def register(body: RegisterRequest):
    # 이미 가입된 이메일인지 확인
    if body.email in fake_users:
        raise HTTPException(status_code=409, detail="이미 사용 중인 이메일입니다.")

    user_id = str(len(fake_users) + 1)
    fake_users[body.email] = {
        "id": user_id,
        "email": body.email,
        "password_hash": hash_password(body.password),  # 비밀번호는 해시로 저장
        "nickname": body.nickname,
    }
    return RegisterResponse(
        user_id=user_id,
        email=body.email,
        nickname=body.nickname,
        created_at=datetime.utcnow()
    )

@router.post("/login", response_model=TokenResponse)
async def login(body: LoginRequest, response: Response):
    user = fake_users.get(body.email)

    # 유저 없거나 비밀번호 틀리면 동일한 에러 (보안상 구분 안 함)
    if not user or not verify_password(body.password, user["password_hash"]):
        raise HTTPException(status_code=401, detail="이메일 또는 비밀번호가 올바르지 않습니다.")

    access_token = create_access_token(user["id"])
    refresh_token = create_refresh_token(user["id"])

    # refresh_token은 HttpOnly 쿠키로 설정 (JS에서 접근 불가 → 보안)
    response.set_cookie(
        key="refresh_token",
        value=refresh_token,
        httponly=True,
        secure=True,
        samesite="strict",
        max_age=60 * 60 * 24 * 7  # 7일
    )

    return TokenResponse(access_token=access_token)

@router.post("/refresh", response_model=TokenResponse)
async def refresh(refresh_token: str = Cookie(None)):
    if not refresh_token:
        raise HTTPException(status_code=401, detail="refresh_token이 없습니다.")

    payload = decode_token(refresh_token)
    if not payload or payload.get("type") != "refresh":
        raise HTTPException(status_code=401, detail="유효하지 않은 토큰입니다.")

    new_access_token = create_access_token(payload["sub"])
    return TokenResponse(access_token=new_access_token)

@router.post("/logout")
async def logout(response: Response):
    response.delete_cookie("refresh_token")
    return {"message": "logged out"}
```

---

### 튜토리얼: 서버 실행 및 테스트

`app/main.py`:

```python
from fastapi import FastAPI
from app.api.v1 import auth

app = FastAPI(title="Phishing Detector API")
app.include_router(auth.router)

@app.get("/health")
async def health():
    return {"status": "ok"}
```

```bash
# 서버 실행
uvicorn app.main:app --reload

# 브라우저에서 http://localhost:8000/docs 열면
# Swagger UI로 API 바로 테스트 가능!
```

> `--reload` 옵션: 코드 수정하면 서버가 자동으로 재시작돼요. 개발할 때만 써요.

---

## Android

### 목표
- 프로젝트 세팅
- 로그인 / 회원가입 화면 구현
- Retrofit2로 Auth API 호출

---

### 원리 설명: Retrofit2가 뭔가요?

Android에서 HTTP 요청(API 호출)을 쉽게 할 수 있게 해주는 라이브러리예요.

```kotlin
// Retrofit 없이 하면 이렇게 복잡해요
val url = URL("https://api.example.com/login")
val conn = url.openConnection() as HttpURLConnection
conn.requestMethod = "POST"
// ... 10줄 더 ...

// Retrofit 쓰면 이렇게 간단해요
val response = apiService.login(LoginRequest(email, password))
```

---

### 튜토리얼: 의존성 추가

`build.gradle (app)`:

```gradle
dependencies {
    // Retrofit (HTTP 통신)
    implementation 'com.squareup.retrofit2:retrofit:2.11.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.11.0'

    // Coroutines (비동기 처리)
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.0'

    // ViewModel (화면 로직 분리)
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.8.0'
}
```

---

### 튜토리얼: API 인터페이스 정의

```kotlin
// data/remote/ApiService.kt

data class LoginRequest(val email: String, val password: String)
data class TokenResponse(val access_token: String, val token_type: String)

interface ApiService {
    @POST("api/v1/auth/login")
    suspend fun login(@Body request: LoginRequest): Response<TokenResponse>

    @POST("api/v1/auth/register")
    suspend fun register(@Body request: RegisterRequest): Response<RegisterResponse>
}

// Retrofit 인스턴스 생성
object RetrofitClient {
    private const val BASE_URL = "http://10.0.2.2:8000/"
    // 10.0.2.2 = 에뮬레이터에서 내 PC localhost를 가리키는 주소

    val api: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}
```

---

### 튜토리얼: 로그인 ViewModel

```kotlin
// ui/auth/LoginViewModel.kt

class LoginViewModel : ViewModel() {

    // UI 상태 (로딩중 / 성공 / 실패)
    private val _loginState = MutableLiveData<LoginState>()
    val loginState: LiveData<LoginState> = _loginState

    fun login(email: String, password: String) {
        viewModelScope.launch {  // 비동기로 실행
            _loginState.value = LoginState.Loading

            try {
                val response = RetrofitClient.api.login(LoginRequest(email, password))

                if (response.isSuccessful) {
                    val token = response.body()!!.access_token
                    // 토큰 저장 (SharedPreferences)
                    saveToken(token)
                    _loginState.value = LoginState.Success
                } else {
                    _loginState.value = LoginState.Error("이메일 또는 비밀번호를 확인해주세요.")
                }
            } catch (e: Exception) {
                _loginState.value = LoginState.Error("네트워크 오류가 발생했습니다.")
            }
        }
    }
}

sealed class LoginState {
    object Loading : LoginState()
    object Success : LoginState()
    data class Error(val message: String) : LoginState()
}
```

---

---

# 4주차 — 카메라 스캔 + Scan API 스텁

## 백엔드

### 목표
- `POST /api/v1/scan/url` 스텁 완성
- 실제 ML/VT 없이 고정 응답 반환 (Android 팀이 연동 시작할 수 있게)

---

### 원리 설명: 스텁(Stub)이 뭔가요?

클라우드팀 서버가 아직 안 만들어졌을 때,
가짜 고정 응답을 내려주는 임시 엔드포인트예요.

Android 팀은 진짜 ML 결과 없이도 화면 개발을 시작할 수 있어요.

```python
# 스텁: 항상 같은 응답 반환
@router.post("/scan/url")
async def scan_url_stub(body: ScanRequest):
    return {
        "scan_id": str(uuid4()),
        "url": body.url,
        "risk_level": "suspicious",
        "final_score": 55.0,
        "score_breakdown": { ... },
        "xai_features": [ ... ],
        "scanned_at": datetime.utcnow().isoformat()
    }
```

---

### 튜토리얼: Scan API 스텁

`app/schemas/scan.py`:

```python
from pydantic import BaseModel, HttpUrl
from typing import List, Literal
from datetime import datetime

class ScanRequest(BaseModel):
    url: str
    source: Literal["camera", "manual"] = "manual"

class VTBreakdown(BaseModel):
    detected_engines: int
    total_engines: int
    weighted_score: float

class MLBreakdown(BaseModel):
    probability: float
    weighted_score: float

class ScoreBreakdown(BaseModel):
    weighting_case: Literal["A", "B"]
    vt: VTBreakdown
    ml: MLBreakdown

class XAIFeature(BaseModel):
    feature: str
    value: float
    contribution: float
    direction: Literal["danger", "safe"]

class ScanResponse(BaseModel):
    scan_id: str
    url: str
    cached: bool
    risk_level: Literal["safe", "suspicious", "dangerous"]
    final_score: float
    weighting_case: Literal["A", "B"]
    threshold_message: str
    score_breakdown: ScoreBreakdown
    xai_features: List[XAIFeature]
    scanned_at: str
```

`app/api/v1/scan.py`:

```python
from fastapi import APIRouter, Depends
from uuid import uuid4
from datetime import datetime
from app.schemas.scan import ScanRequest, ScanResponse
from app.core.dependencies import get_current_user

router = APIRouter(prefix="/api/v1/scan", tags=["scan"])

def get_threshold_message(score: float) -> str:
    if score < 40:
        return "안전한 사이트입니다."
    elif score < 70:
        return "신규 생성된 사이트로 의심됩니다. 주의하세요."
    else:
        return "큐싱 사이트로 확정됩니다. 접속이 차단되었습니다."

@router.post("/url", response_model=ScanResponse)
async def scan_url(body: ScanRequest, current_user=Depends(get_current_user)):
    # 4주차: 스텁 (고정 응답)
    # 5주차에 클라우드팀 Lambda 연동으로 교체 예정

    final_score = 55.0  # 임시 고정값

    return ScanResponse(
        scan_id=str(uuid4()),
        url=body.url,
        cached=False,
        risk_level="suspicious",
        final_score=final_score,
        weighting_case="B",
        threshold_message=get_threshold_message(final_score),
        score_breakdown={
            "weighting_case": "B",
            "vt": {"detected_engines": 0, "total_engines": 90, "weighted_score": 0.0},
            "ml": {"probability": 0.687, "weighted_score": 55.0}
        },
        xai_features=[
            {"feature": "url_length", "value": 142, "contribution": 0.21, "direction": "danger"},
            {"feature": "domain_age_days", "value": 3, "contribution": 0.35, "direction": "danger"},
        ],
        scanned_at=datetime.utcnow().isoformat()
    )
```

---

### 원리 설명: Depends(get_current_user)가 뭔가요?

엔드포인트마다 "로그인한 유저만 접근 가능"을 적용하는 방법이에요.

```python
# app/core/dependencies.py

from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from app.core.security import decode_token

security = HTTPBearer()

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials  # Header의 "Bearer {토큰}"에서 토큰 추출
    payload = decode_token(token)

    if not payload:
        raise HTTPException(status_code=401, detail="유효하지 않은 토큰입니다.")

    return payload["sub"]  # user_id 반환
```

`Depends(get_current_user)`를 붙이면 토큰 없는 요청은 자동으로 401 에러가 나요.

---

## Android

### 목표
- CameraX로 QR 코드 스캔
- ML Kit으로 URL 텍스트 추출
- 스캔 결과 화면 UI 구성

---

### 원리 설명: CameraX + ML Kit 흐름

```
카메라 미리보기 (CameraX)
        ↓
프레임마다 이미지 캡처
        ↓
ML Kit QR 스캐너에 넣기
        ↓
QR 코드 감지되면 URL 추출
        ↓
Retrofit으로 백엔드 API 호출
```

---

### 튜토리얼: CameraX + ML Kit 세팅

```gradle
dependencies {
    // CameraX
    implementation 'androidx.camera:camera-camera2:1.3.3'
    implementation 'androidx.camera:camera-lifecycle:1.3.3'
    implementation 'androidx.camera:camera-view:1.3.3'

    // ML Kit QR 스캐너
    implementation 'com.google.mlkit:barcode-scanning:17.3.0'
}
```

```kotlin
// ui/scan/ScanFragment.kt (핵심 부분)

class ScanFragment : Fragment() {

    private lateinit var cameraProvider: ProcessCameraProvider
    private val barcodeScanner = BarcodeScanning.getClient()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        startCamera()
    }

    private fun startCamera() {
        val cameraProviderFuture = ProcessCameraProvider.getInstance(requireContext())

        cameraProviderFuture.addListener({
            cameraProvider = cameraProviderFuture.get()

            // 미리보기 설정
            val preview = Preview.Builder().build().also {
                it.setSurfaceProvider(binding.previewView.surfaceProvider)
            }

            // 이미지 분석 설정 (프레임마다 QR 스캔)
            val imageAnalyzer = ImageAnalysis.Builder().build().also {
                it.setAnalyzer(ContextCompat.getMainExecutor(requireContext())) { imageProxy ->
                    scanQRCode(imageProxy)
                }
            }

            cameraProvider.bindToLifecycle(
                viewLifecycleOwner,
                CameraSelector.DEFAULT_BACK_CAMERA,
                preview,
                imageAnalyzer
            )
        }, ContextCompat.getMainExecutor(requireContext()))
    }

    private fun scanQRCode(imageProxy: ImageProxy) {
        val mediaImage = imageProxy.image ?: return

        val inputImage = InputImage.fromMediaImage(mediaImage, imageProxy.imageInfo.rotationDegrees)

        barcodeScanner.process(inputImage)
            .addOnSuccessListener { barcodes ->
                barcodes.firstOrNull()?.rawValue?.let { url ->
                    // URL 감지! 백엔드 호출
                    viewModel.scanUrl(url)
                    cameraProvider.unbindAll()  // 스캔 중지
                }
            }
            .addOnCompleteListener {
                imageProxy.close()  // 반드시 닫아야 다음 프레임 처리 가능
            }
    }
}
```

---

---

# 5주차 — Scan API 완성 + 결과 화면

## 백엔드

### 목표
- 클라우드팀 Lambda 연동 (스텁 → 실제 호출로 교체)
- 에러 처리 완성

---

### 원리 설명: 클라우드팀 Lambda를 어떻게 호출하나요?

클라우드팀이 Lambda 앞에 API Gateway URL을 줄 거예요.
백엔드는 그 URL로 HTTP 요청을 보내면 돼요.

```python
# app/services/scoring.py

import httpx  # 비동기 HTTP 클라이언트

LAMBDA_URL = "https://xxxxx.execute-api.ap-northeast-2.amazonaws.com/prod/scan"

async def get_score_from_lambda(url: str) -> dict:
    async with httpx.AsyncClient(timeout=10.0) as client:
        response = await client.post(
            LAMBDA_URL,
            json={"url": url},
            headers={"X-API-Key": settings.LAMBDA_API_KEY}
        )
        response.raise_for_status()
        return response.json()
        # 반환값: { final_score, risk_level, score_breakdown, xai_features, ... }
```

이렇게 하면 복잡한 ML/VT 계산은 클라우드팀이 다 해주고,
백엔드는 결과를 받아서 Android에 전달만 하면 돼요.

---

### 튜토리얼: 스텁 → 실제 Lambda 연동으로 교체

```python
# app/api/v1/scan.py (5주차 버전)

@router.post("/url", response_model=ScanResponse)
async def scan_url(body: ScanRequest, current_user=Depends(get_current_user)):

    # 1. Redis 캐시 확인 (클라우드팀이 제공하는 Redis 또는 직접 확인)
    url_hash = hashlib.sha256(body.url.encode()).hexdigest()

    # 2. Lambda 호출 (클라우드팀 서버)
    try:
        lambda_result = await get_score_from_lambda(body.url)
    except httpx.TimeoutException:
        raise HTTPException(status_code=504, detail="분석 서버 응답 시간이 초과됐습니다.")
    except httpx.HTTPStatusError as e:
        raise HTTPException(status_code=502, detail="분석 서버 오류가 발생했습니다.")

    # 3. 응답 포장해서 반환
    return ScanResponse(
        scan_id=str(uuid4()),
        url=body.url,
        cached=lambda_result.get("cached", False),
        risk_level=lambda_result["risk_level"],
        final_score=lambda_result["final_score"],
        weighting_case=lambda_result["weighting_case"],
        threshold_message=get_threshold_message(lambda_result["final_score"]),
        score_breakdown=lambda_result["score_breakdown"],
        xai_features=lambda_result["xai_features"],
        scanned_at=datetime.utcnow().isoformat()
    )
```

---

## Android

### 목표
- 결과 화면: 점수 게이지, 위험 단계 색상, XAI 피처 목록

---

### 튜토리얼: 결과 화면 ViewModel

```kotlin
// ui/result/ResultViewModel.kt

class ResultViewModel : ViewModel() {

    private val _scanResult = MutableLiveData<ScanResultState>()
    val scanResult: LiveData<ScanResultState> = _scanResult

    fun scanUrl(url: String) {
        viewModelScope.launch {
            _scanResult.value = ScanResultState.Loading

            try {
                val token = TokenManager.getToken()  // 저장된 토큰 가져오기
                val response = RetrofitClient.api.scanUrl(
                    "Bearer $token",
                    ScanRequest(url = url, source = "camera")
                )

                if (response.isSuccessful) {
                    _scanResult.value = ScanResultState.Success(response.body()!!)
                } else {
                    _scanResult.value = ScanResultState.Error("분석에 실패했습니다.")
                }
            } catch (e: Exception) {
                _scanResult.value = ScanResultState.Error("네트워크 오류가 발생했습니다.")
            }
        }
    }
}
```

---

### 튜토리얼: 위험 단계에 따른 UI 분기

```kotlin
// ui/result/ResultFragment.kt

private fun updateUI(result: ScanResponse) {
    binding.scoreText.text = "${result.final_score.toInt()}점"

    // 위험 단계별 색상 + 메시지
    when (result.risk_level) {
        "safe" -> {
            binding.riskBadge.setBackgroundColor(Color.parseColor("#4CAF50"))  // 초록
            binding.riskLabel.text = "안전"
        }
        "suspicious" -> {
            binding.riskBadge.setBackgroundColor(Color.parseColor("#FFC107"))  // 노랑
            binding.riskLabel.text = "의심"
        }
        "dangerous" -> {
            binding.riskBadge.setBackgroundColor(Color.parseColor("#F44336"))  // 빨강
            binding.riskLabel.text = "위험"
            showBlockDialog()  // 접속 차단 다이얼로그
        }
    }

    binding.thresholdMessage.text = result.threshold_message

    // XAI 피처 목록 표시
    val adapter = XAIFeatureAdapter(result.xai_features)
    binding.xaiRecyclerView.adapter = adapter
}

private fun showBlockDialog() {
    AlertDialog.Builder(requireContext())
        .setTitle("접속 차단")
        .setMessage("이 사이트는 피싱 사이트로 확인되었습니다.")
        .setPositiveButton("확인") { _, _ -> findNavController().navigateUp() }
        .setCancelable(false)
        .show()
}
```

---

---

# 6주차 — 방문기록 / 즐겨찾기

## 백엔드

### 목표
- `GET /api/v1/users/me/visits`
- `GET/POST/DELETE /api/v1/users/me/bookmarks`

---

### 튜토리얼: 방문기록 API

```python
# app/api/v1/users.py

router = APIRouter(prefix="/api/v1/users", tags=["users"])

@router.get("/me/visits")
async def get_visits(
    page: int = 1,
    size: int = 20,
    current_user: str = Depends(get_current_user)
):
    # DB에서 현재 유저의 방문기록 조회 (나중에 실제 DB 쿼리로 교체)
    return {
        "items": [
            {
                "url": "https://example.com",
                "risk_level": "safe",
                "visited_at": datetime.utcnow().isoformat()
            }
        ],
        "total": 1,
        "page": page,
        "size": size
    }

@router.get("/me/bookmarks")
async def get_bookmarks(current_user: str = Depends(get_current_user)):
    return {"items": []}

@router.post("/me/bookmarks", status_code=201)
async def add_bookmark(body: BookmarkRequest, current_user: str = Depends(get_current_user)):
    return {"bookmark_id": str(uuid4()), "created_at": datetime.utcnow().isoformat()}

@router.delete("/me/bookmarks/{bookmark_id}")
async def delete_bookmark(bookmark_id: str, current_user: str = Depends(get_current_user)):
    return {"message": "deleted"}
```

---

---

# 7주차 — 신고 + 블랙리스트

## 백엔드

### 목표
- `POST /api/v1/report`
- `GET /api/v1/report/blacklist`

---

### 튜토리얼: 신고 API

```python
# app/api/v1/report.py

@router.post("/report", status_code=202)
async def report_url(body: ReportRequest, current_user: str = Depends(get_current_user)):
    # DB에 신고 내역 저장 (status: pending)
    # 클라우드팀 Lambda Verifier가 비동기로 VT 교차검증 처리

    return {
        "report_id": str(uuid4()),
        "status": "queued",
        "message": "검토 후 블랙리스트에 반영됩니다."
    }

@router.get("/report/blacklist")
async def get_blacklist(page: int = 1, size: int = 20):
    # 인증 불필요 (공개 조회)
    return {
        "items": [
            {
                "rank": 1,
                "url": "https://phishing-example.com",
                "report_count": 42,
                "risk_level": "dangerous",
                "final_score": 91.2,
                "last_reported_at": datetime.utcnow().isoformat()
            }
        ],
        "total": 1,
        "page": page,
        "size": size
    }
```

---

---

# 8주차 — 로그 관리 (캘린더)

## 백엔드

### 목표
- `GET /api/v1/users/me/scan-logs` (날짜별 집계 포함)
- `GET /api/v1/users/me/scan-logs/{scan_id}/detail`

---

### 튜토리얼: 캘린더 집계 응답

```python
@router.get("/me/scan-logs")
async def get_scan_logs(
    from_date: str = None,
    to_date: str = None,
    current_user: str = Depends(get_current_user)
):
    return {
        "summary": {
            "total_scans": 87,
            "safe_count": 61,
            "suspicious_count": 18,
            "dangerous_count": 8
        },
        "calendar": {
            "2025-03-10": {"count": 3, "dangerous": 1, "avg_score": 45.2},
            "2025-03-11": {"count": 5, "dangerous": 0, "avg_score": 18.0}
        },
        "logs": [
            {
                "scan_id": str(uuid4()),
                "url": "https://example.com",
                "risk_level": "safe",
                "final_score": 12.4,
                "scanned_at": datetime.utcnow().isoformat()
            }
        ]
    }
```

---

---

# 9주차 — 통합 테스트 + 에러 처리

## 백엔드

### 원리 설명: 에러 처리가 왜 중요한가요?

Android가 서버에 요청했을 때 뭔가 잘못되면,
무슨 에러인지 알 수 있는 형식으로 응답해야 해요.

---

### 튜토리얼: 전역 에러 핸들러

```python
# app/main.py에 추가

from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

@app.exception_handler(RequestValidationError)
async def validation_error_handler(request, exc):
    # Pydantic 검증 실패 (필드 형식 오류 등)
    return JSONResponse(
        status_code=422,
        content={
            "error": "VALIDATION_ERROR",
            "message": "요청 형식이 올바르지 않습니다.",
            "detail": str(exc)
        }
    )

@app.exception_handler(HTTPException)
async def http_error_handler(request, exc):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": "HTTP_ERROR",
            "message": exc.detail,
            "status_code": exc.status_code
        }
    )
```

---

### 튜토리얼: pytest로 API 테스트

```bash
pip install pytest httpx pytest-asyncio
```

```python
# tests/test_auth.py

import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_register():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/api/v1/auth/register", json={
            "email": "test@example.com",
            "password": "password123",
            "nickname": "테스터"
        })
    assert response.status_code == 201
    assert response.json()["email"] == "test@example.com"

@pytest.mark.asyncio
async def test_login_wrong_password():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/api/v1/auth/login", json={
            "email": "test@example.com",
            "password": "틀린비밀번호"
        })
    assert response.status_code == 401
```

```bash
# 테스트 실행
pytest tests/ -v
```

---

---

# 10~15주차 — 마무리

## 10주차: 성능 + 보안 점검

- 모든 API 응답시간 측정 (목표: 스캔 API 3초 이내)
- SQL injection 방지 확인 (SQLAlchemy ORM 쓰면 자동 방지)
- JWT 만료 처리 Android에서 정상 동작하는지 확인

## 11~13주차: 클라우드팀 연동

- 스텁으로 만든 Scan API를 실제 Lambda URL로 교체
- 실제 DB(RDS) 연결로 교체
- Redis 캐시 연결

## 14~15주차: 발표 준비

- 앱 시연 시나리오 작성
- 실제 피싱 URL 예시로 테스트
- 버그 최종 수정

---

---

# 빠른 참고표

## 에러 코드 의미

| 코드 | 의미 | 대처 |
|------|------|------|
| 200 | 성공 | — |
| 201 | 생성 성공 | — |
| 202 | 접수됨 (비동기 처리 중) | — |
| 400 | 요청 형식 오류 | 요청 JSON 확인 |
| 401 | 인증 실패 | 토큰 확인 / 재로그인 |
| 409 | 중복 (이미 존재) | 이메일 중복 등 |
| 422 | Pydantic 검증 실패 | 필드 형식 확인 |
| 502 | 클라우드팀 서버 오류 | Lambda 확인 |
| 504 | 클라우드팀 서버 타임아웃 | Lambda 확인 |

## Android ↔ 백엔드 통신 체크리스트

- [ ] Header에 `Authorization: Bearer {token}` 붙이기
- [ ] Content-Type: application/json 확인
- [ ] 응답 필드명 오타 없는지 확인 (snake_case)
- [ ] 에러 응답 처리 (401, 422 등)
- [ ] 네트워크 없을 때 처리


---

## 현재 완료된 것

```
Phase 0 — 환경 세팅 ✅
  ✅ GitHub 리포 생성 (Q-Guarder-App, Private)
  ✅ Android 프로젝트 초기 세팅
  ✅ libs.versions.toml + build.gradle 설정
  ✅ 패키지 구조 생성 (ui/data/domain/di)
  ✅ nav_graph.xml + bottom_nav_menu.xml
  ✅ Fragment 11개 + XML 레이아웃 전체
  ✅ Domain Model (RiskLevel, ScanResult 등)
  ✅ AppDatabase (Room — Whitelist, FavoritesCache)
  ✅ TokenManager (EncryptedSharedPreferences)
  ✅ AuthRepository
  ✅ ApiService.kt (전체 엔드포인트)
  ✅ NetworkModule + DatabaseModule (Hilt DI)
  ✅ AuthInterceptor (토큰 자동 주입)
  ✅ AndroidManifest.xml (권한 추가)
  ✅ develop → main merge 완료

feature/login 진행 중 🔄
  ✅ 화면 이동 (스플래시→로그인→회원가입) 확인
  ✅ 회원가입 서버 연동 성공
  ❌ 로그인 500 에러 (클라우드팀 수정 중)
  ⬜ 로그인 성공 후 ScanFragment 이동
  ⬜ SplashFragment 자동 로그인 분기
  ⬜ ProfileFragment 로그아웃 연동
```

---

## 앞으로의 계획 (4월 말까지)

```
3월 말 — feature/login 마무리
  □ 로그인 연동 성공
  □ 자동 로그인 (SplashFragment)
  □ 로그아웃 연동
  □ develop PR + merge

4월 1주 (3/29~4/4) — feature/scan
  □ ScanFragment CameraX 실제 QR 인식 테스트
  □ ScanViewModel 캐시 히트 로직 확인
     (Whitelist → Favorites → 서버 순서)
  □ ScanningFragment 서버 연동
     (POST /api/v1/scan/url)
  □ ResultWhitelistFragment (캐시 히트 결과)
  □ ResultSecuredFragment (SECURED/SUSPICIOUS)
  □ ResultBlockedFragment (BLOCKED)
  □ develop PR + merge

4월 2주 (4/5~4/11) — feature/logs
  □ LogsFragment 서버에서 스캔 이력 조회
     (GET /api/v1/users/me/scan-logs)
  □ 통계 카드 (안전/위험 건수)
  □ 신고하기 버튼 연동
     (POST /api/v1/report)
  □ develop PR + merge

4월 3주 (4/12~4/18) — feature/favorites
  □ FavoritesFragment 즐겨찾기 목록
     (GET /api/v1/users/me/bookmarks)
  □ 결과 화면에서 즐겨찾기 추가/삭제
     (POST/DELETE /api/v1/users/me/bookmarks)
  □ FavoritesCache Room DB 서버 동기화
  □ develop PR + merge

4월 4주 (4/19~4/25) — feature/profile + UI 폴리싱
  □ ProfileFragment 유저 정보 표시
     (GET /api/v1/users/me)
  □ 로그아웃 실제 동작
  □ Whitelist 관리 화면
  □ 전체 화면 UI 통일 (폰트/여백/색상)
  □ 스플래시 애니메이션 수정
  □ 버튼 클릭 효과 전체 적용
  □ develop → main 최종 merge
```

---

## 우선순위 요약

```
당장 이번 주    로그인 fix 대기 → 로그인/로그아웃 완료
다음 주        QR 스캔 → 결과 화면 (핵심 기능)
4월 2주        스캔 로그 + 신고
4월 3주        즐겨찾기
4월 4주        프로필 + UI 마무리
```

클라우드팀 서버 진행 속도에 따라 스캔 기능이 제일 중요하니까, 로그인 되는 대로 바로 스캔으로 넘어가는 게 좋아요!