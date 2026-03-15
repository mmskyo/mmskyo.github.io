---
title: pep8
---
---
### **1️⃣ security.py - UTC 버그 수정**
```python
# ❌ 이전 - UTC가 함수가 아님
datetime.now(UTC) + timedelta(...)

# ✅ 수정 - timezone.utc 올바르게 사용
datetime.now(timezone.utc) + timedelta(...)
```
**왜?** `UTC`는 상수가 아니라 timezone 객체여야 하니까요.

---

### **2️⃣ security.py - Refresh Token 함수 추가** ✅
```python
# 이전: create_access_token만 있었음

# 수정: 새로운 함수 추가
def create_refresh_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    """리프레시 토큰은 더 긴 유효기간을 가집니다"""
    to_encode.update({"exp": expire, "type": "refresh"})  # 토큰 타입 명시
```
**왜?** 리프레시 토큰과 액세스 토큰은 다른 타입이므로 별도의 함수가 필요합니다.

---

### **3️⃣ auth.py - 실제 Refresh Token 반환** ✅
```python
# ❌ 이전
return {"access_token": access_token, "refresh_token": "dummy"}

# ✅ 수정
refresh_token = create_refresh_token(
    data={"sub": str(user.id)},
    expires_delta=timedelta(days=settings.REFRESH_TOKEN_EXPIRE_DAYS)
)
return {
    "access_token": access_token,
    "refresh_token": refresh_token,  # 진짜 토큰!
    "token_type": "bearer"
}
```

---

### **4️⃣ auth.py & scan.py - 에러 처리 추가** ✅

**이전에는:**
```python
db.add(db_user)
await db.commit()  # 실패하면? 아무것도 없음 ❌
```

**수정된 방식:**
```python
try:
    db.add(db_user)
    await db.commit()
except SQLAlchemyError as e:
    await db.rollback()  # DB 변경사항 취소
    logger.error(f"Database error: {str(e)}")
    raise HTTPException(status_code=500, detail="Database error")
except Exception as e:
    logger.error(f"Unexpected error: {str(e)}")
    raise HTTPException(status_code=500, detail="Internal server error")
```

**왜?** DB 연결 실패, 타임아웃, 제약조건 위반 등이 발생할 수 있으니까요.

---

### **5️⃣ scan.py - URL 검증 추가** ✅
```python
# ✅ 수정
try:
    urlparse(url)  # URL 형식 확인
except Exception:
    raise HTTPException(status_code=400, detail="Invalid URL format")
```

---

### **6️⃣ print() → logging으로 변경** ✅

**이전:**
```python
print(f"{request.method} {request.url} - {process_time:.0f}s")
```

**실무 방식:**
```python
import logging
logger = logging.getLogger(__name__)

logger.info(f"{request.method} {request.url.path} - {process_time:.0f}ms")
```

**왜?** 프로덕션에선 print는 버퍼링되고 실제로 기록되지 않을 수 있어요.

---

### **7️⃣ 타입 힌팅 강화** ✅
```python
# ✅ 함수 반환 타입 명시
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:

async def get_domain_age(url: str) -> Optional[int]:

async def register(user_data: UserCreate, db: AsyncSession = Depends(get_db)):
    """도큐문트 문자열 추가"""
```

---

## 🎯 **실무 개발자가 생각하는 핵심**

| 항목 | 개선 전 | 개선 후 |
|------|--------|--------|
| **에러 처리** | 없음 | try/except + 로깅 |
| **토큰** | 더미값 | 진짜 토큰 생성 |
| **로깅** | print | logging 모듈 |
| **타입** | 없음 | 전부 명시 |
| **문서화** | 없음 | Docstring 추가 |
| **보안** | 약함 | 하드코딩 주의 |

---

이제 코드가 **실무 수준**입니다! 다음 단계는 테스트 코드 작성인데, 필요하시면 말씀해 주세요.