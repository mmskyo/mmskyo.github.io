---
title: logs
parent: pcd
---
---
## 로그 기능 전체 흐름

```
서버 API
  ↓  (HTTP 응답)
LogsViewModel  ←── Hilt가 자동 주입
  ↓  (StateFlow/SharedFlow로 데이터 전달)
LogsFragment   ←── ViewModel 관찰
  ↓  (submitList 호출)
LogsAdapter
  ↓  (아이템마다 bind 호출)
item_log.xml   (화면에 표시)
```

---

## 1. `ScanLogItemDto` — 데이터 모양 정의

```kotlin
data class ScanLogItemDto(
    val scanId:    String,  // 고유 ID
    val url:       String,  // 스캔된 URL
    val riskLevel: String,  // "BLOCKED" / "SECURED" / "SUSPICIOUS"
    val scannedAt: String,  // "2026-04-13T07:26:00" (ISO 형식)
)
```

**왜 필요하냐면** — 서버에서 JSON으로 오는 데이터를 Kotlin 객체로 변환해서 쓰려면 그 모양을 미리 정의해야 합니다. 서버가 `{ "scan_id": "1", "url": "...", ... }` 이렇게 보내면, Retrofit + Gson이 자동으로 이 클래스에 채워줍니다.

---

## 2. `LogsViewModel` — 데이터 관리자

Fragment는 화면을 그리는 역할만 해야 하고, 데이터를 직접 다루면 안 됩니다. 그래서 ViewModel이 중간에서 데이터를 가져오고 관리합니다.

```kotlin
@HiltViewModel  // Hilt가 이 클래스를 자동으로 만들어줌
class LogsViewModel @Inject constructor(
    private val apiService: ApiService  // 서버 통신 도구를 자동 주입받음
) : ViewModel() {
```

**`@HiltViewModel` + `@Inject constructor`가 왜 필요하냐면** — ViewModel은 `new LogsViewModel()`로 직접 만들면 안 됩니다. 화면 회전 같은 상황에서도 같은 인스턴스를 유지해야 하기 때문에, Android가 직접 생성·관리합니다. Hilt가 `apiService`를 어디서 가져올지 알아서 연결해줍니다.

---

### StateFlow vs SharedFlow — 왜 두 종류를 쓰나요?

```kotlin
// StateFlow — 현재 값을 항상 갖고 있음
private val _logs  = MutableStateFlow<List<ScanLogItemDto>>(emptyList())
private val _stats = MutableStateFlow<ScanSummaryDto?>(null)

// SharedFlow — 값을 저장 안 하고 이벤트처럼 한 번만 쏨
private val _error = MutableSharedFlow<String>()
```

||StateFlow|SharedFlow|
|---|---|---|
|초기값|있음 (`emptyList()`, `null`)|없음|
|언제 씀|목록, 통계처럼 **화면에 계속 보여야 하는 데이터**|에러 토스트처럼 **한 번만 보여야 하는 이벤트**|
|Fragment가 늦게 구독해도|현재 값을 즉시 받음|이미 지난 이벤트는 못 받음|

StateFlow로 에러를 관리하면 화면 회전 시 에러 토스트가 다시 뜨는 문제가 생깁니다. SharedFlow는 그런 문제가 없습니다.

---

### `loadLogs()` 동작 원리

```kotlin
fun loadLogs() {
    viewModelScope.launch {   // 백그라운드에서 실행 (메인 스레드 안 막음)
        _isLoading.value = true
        try {
            val response = apiService.getScanLogs()  // 서버에 HTTP GET 요청
            if (response.isSuccessful) {
                _logs.value  = body?.logs    // StateFlow 값 업데이트
                _stats.value = body?.summary // → Fragment가 자동으로 감지
            }
        } catch (e: Exception) {
            loadDummyData()  // 서버 없으면 더미 데이터
        } finally {
            _isLoading.value = false  // 성공/실패 상관없이 항상 실행
        }
    }
}
```

**`viewModelScope.launch`가 왜 필요하냐면** — 네트워크 통신은 시간이 걸립니다. 메인 스레드에서 직접 실행하면 앱이 그동안 멈춥니다(ANR). `launch`를 쓰면 백그라운드에서 실행되고, ViewModel이 사라질 때 자동으로 취소됩니다.

---

## 3. `LogsFragment` — 화면 담당

```kotlin
private val viewModel: LogsViewModel by viewModels()
```

**`by viewModels()`가 왜 필요하냐면** — 직접 `LogsViewModel()`로 만들면 Fragment가 재생성될 때마다 새 ViewModel이 만들어져서 데이터가 날아갑니다. `by viewModels()`는 Android가 관리하는 ViewModel을 받아오므로, 화면 회전 시에도 동일한 인스턴스와 데이터가 유지됩니다.

---

### `repeatOnLifecycle`이 왜 필요하냐면

```kotlin
viewLifecycleOwner.lifecycleScope.launch {
    viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
        launch { viewModel.logs.collect { ... } }
        launch { viewModel.stats.collect { ... } }
        launch { viewModel.error.collect { ... } }
    }
}
```

Fragment는 백스택에 쌓여있는 동안 뷰가 없어도 존재합니다. `repeatOnLifecycle(STARTED)`가 없으면:

- 다른 화면으로 이동해도 백그라운드에서 계속 데이터를 받아 처리하려 함
- 뷰가 이미 사라진 상태에서 `binding.*`에 접근 → **크래시**

`STARTED` 상태(화면이 실제로 보이는 상태)일 때만 collect하고, 화면이 안 보이면 자동으로 멈춥니다.

`launch`를 3개 쓰는 이유는 `logs`, `stats`, `error`를 **동시에** collect하기 위해서입니다. 하나의 `launch`로 순서대로 collect하면 첫 번째가 끝날 때까지 두 번째는 시작도 못 합니다.

---

## 4. `LogsAdapter` — 목록 렌더링

```kotlin
class LogsAdapter(
    private val onItemClick: (ScanLogItemDto) -> Unit
) : ListAdapter<ScanLogItemDto, LogsAdapter.LogViewHolder>(DiffCallback)
```

**`ListAdapter`를 쓰는 이유** — 일반 `RecyclerView.Adapter`는 `notifyDataSetChanged()`를 호출하면 목록 전체를 다시 그립니다. `ListAdapter`는 `DiffCallback`으로 **뭐가 바뀌었는지만 계산해서** 그 부분만 업데이트합니다. 목록이 100개면 바뀐 1개만 다시 그립니다.

---

### DiffCallback이 왜 2개의 메서드를 갖나요?

```kotlin
// 1단계: 같은 항목인가? (scanId로 판단)
override fun areItemsTheSame(old, new) = old.scanId == new.scanId

// 2단계: 내용이 같은가? (모든 필드 비교)
override fun areContentsTheSame(old, new) = old == new
```

먼저 `areItemsTheSame`으로 "이 항목이 목록에 있는 항목과 같은 건가?"를 확인합니다. 같으면 `areContentsTheSame`으로 내용까지 바뀌었는지 확인합니다. 내용도 같으면 다시 그리지 않습니다. 이 두 단계 덕분에 효율적인 업데이트가 가능합니다.

---

### ViewHolder 패턴이 왜 필요한가요?

```kotlin
inner class LogViewHolder(private val binding: ItemLogBinding) :
    RecyclerView.ViewHolder(binding.root) {
    fun bind(item: ScanLogItemDto) { ... }
}
```

RecyclerView는 화면에 보이는 카드만 만들고, 스크롤할 때 사라지는 카드를 **재사용**합니다. ViewHolder는 카드 하나의 뷰를 잡고 있는 객체입니다. 새 데이터로 `bind()`만 호출하면 기존 카드 뷰를 재활용할 수 있습니다. 카드를 매번 새로 만들면 스크롤이 버벅입니다.

---

### `item_log.xml`과 `fragment_logs.xml`의 차이

```
fragment_logs.xml          ← 화면 전체 틀
├── 통계 카드 (tv_safe_percent, tv_stats_summary...)
├── "상세 검사 로그" 타이틀
├── RecyclerView (rv_logs)  ← 여기 안에 item_log.xml이 반복됨
│    ├── item_log.xml (1번 카드)
│    ├── item_log.xml (2번 카드)
│    └── item_log.xml (3번 카드...)
└── layoutEmpty (로그 없을 때 표시)
```

`fragment_logs.xml`은 화면의 뼈대이고, `item_log.xml`은 리스트 한 줄의 디자인입니다. Adapter가 `item_log.xml`을 데이터 수만큼 복사해서 RecyclerView 안에 채워넣습니다.

---

## 전체 연결 순서 (앱 실행 시)

```
1. 사용자가 "보안 로그" 탭 탭
         ↓
2. LogsFragment 생성
   → by viewModels()로 LogsViewModel 가져옴
         ↓
3. LogsViewModel init 블록 실행
   → loadLogs() 호출
   → viewModelScope.launch로 백그라운드 실행
   → apiService.getScanLogs() HTTP 요청
   → 실패 시 loadDummyData() 호출
   → _logs.value = [더미 5개]
   → _stats.value = ScanSummaryDto(...)
         ↓
4. _logs, _stats 값이 바뀜
   → StateFlow가 "값 바뀌었어요" 신호 전송
         ↓
5. LogsFragment의 collect 블록이 감지
   → logsAdapter.submitList(list) 호출
   → DiffCallback으로 변경 계산
   → RecyclerView에 카드 5개 표시
   → tv_safe_percent, tv_stats_summary 업데이트
```