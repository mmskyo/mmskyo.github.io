---
title: architecture
nav_order: 3
parent: PCD
published: "false"
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
	val riskLevel: RiskLevel, // 아
	
)
```
