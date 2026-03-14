---
title: exapi
nav_order: 7
parent: Projects
---
---
whoisapi 등 외부 서비스 연동

```
qr-safe-scanner/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── security.py
│   │   └── cache.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   └── scan.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── scan.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── scan.py
│   └── db/
│       ├── __init__.py
│       └── session.py
├── requirements.txt
├── .env
└── .gitignore
```

import 할 때 경로 설정 제대로!\
from .core.config X
from app.core.config import settings X
만약 db에서 쓴다면
from core.config O 
why? app.을 붙이는건 자기 자신을 호출하는것!
.. 또한 마찬가지
같은 디렉토리에 이미 존재하기에 다른 폴더라도 붙일 필요가 없다. 그냥 폴더 이름만


class baseurl ? 그거 이메일 어쩌구 못한다그러면
pip install 'pydantic[email]'

서버 실행 명령어
```
fastapi dev app/main.py
```

## 패스워드 72바이트 제한 우회
security.py

```import hashlib

import bcrypt


def verify_password(plain_password: str, hashed_password: str) -> bool:

# 입력 패스워드 → SHA256 → bcrypt 검증

pre_hashed = hashlib.sha256(plain_password.encode('utf-8')).digest()

return bcrypt.checkpw(pre_hashed, hashed_password.encode('utf-8'))

  

def get_password_hash(password: str) -> str:

# 임의 길이 패스워드 → SHA256(72바이트) → bcrypt 해싱

pre_hashed = hashlib.sha256(password.encode('utf-8')).digest()

return bcrypt.hashpw(pre_hashed, bcrypt.gensalt()).decode('utf-8')
```

# UTC 함수 오류
```
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):

to_encode = data.copy() # 토큰에 담을 데이터 복사

if expires_delta:

expire = datetime.now(UTC) + expires_delta # UTC 시간 사용

else:

expire = datetime.now(UTC) + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES) # UTC() 아님!! UTC만

to_encode.update({"exp": expire, "type": "access"}) # 토큰 유형 명시

encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm="HS256") # HS256 알고리즘으로 JWT 생성

return encoded_jwt
```
# 서버 정상 작동 확인
## 1. /docs서버로 들어가서 swaggerui뜨는지 확인
## 2. 회원가입
서버 응답 200번대로 뜨면 성공
## 3. 로그인
200
## 4. 스캔
"image_base64": "test_image_data",
  "extracted_url": "https://naver.com",
  "device_info": "iPhone 15 Pro"
  로 입력
  safe=True 면 성공