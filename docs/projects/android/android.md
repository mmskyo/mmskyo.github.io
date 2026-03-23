---
title: android
parent: android
nav_order: 2
---
---
하지만 **실무(팀 프로젝트) 기준**으로는 로그인 화면을 '무작정 먼저 그리기'보다, **앱 전체의 지도(Navigation)와 뼈대**를 먼저 잡는 것이 정석입니다. 왜냐하면 유저가 이미 로그인한 상태라면 로그인 화면을 건너뛰고 바로 메인(대시보드)으로 가야 하기 때문이죠.

실무에서 진행하는 **체계적인 개발 순서 4단계**를 튜토리얼로 정리해 드립니다.

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
- 