---
title: logs
parent: pcd
---
---
# 로그(기록 + 통계) 기능 구현

## Context

보안 스캔 대시보드에 기록(로그 목록)과 통계 표시 기능 구현.

현재 상태: API/DTO/네비게이션/레이아웃 뼈대는 있지만, **어댑터/데이터 바인딩/에러 핸들링**이 없어서 실제 데이터가 화면에 표시되지 않음.

## 생성 파일 (2개)

### 1. `item_log.xml` 생성

- `app/src/main/res/layout/item_log.xml`

- `item_favorite.xml` 패턴 참고 (MaterialCardView + 가로 LinearLayout)

- 삭제 버튼 없음, 대신 날짜 텍스트 추가

- 구성: `tv_url` (URL), `tv_date` (스캔 날짜), `tv_risk_badge` (위험도 뱃지)

### 2. `LogsAdapter.kt` 생성

- `app/src/main/java/com/qguarder/android/ui/logs/LogsAdapter.kt`

- `FavoritesAdapter` 패턴 그대로 (ListAdapter + DiffUtil)

- `onItemClick: (ScanLogItemDto) -> Unit` 콜백

- `bind()`:

- `tv_url.text = item.url`

- `tv_date.text` = `scannedAt` 파싱 → `"yyyy.MM.dd HH:mm"` 형식

- riskLevel별 뱃지:

- `SECURED` → "안전", 초록 `#1D9E75`, `bg_badge_green`

- `SUSPICIOUS` → "의심", 주황 `#D97706`, `bg_badge_amber` (신규)

- `BLOCKED` → "위험", 빨강 `#E24B4A`, `bg_badge_red`

- DiffCallback: `areItemsTheSame` = scanId 비교

### (추가) `bg_badge_amber.xml` 생성

- `app/src/main/res/drawable/bg_badge_amber.xml`

- `bg_badge_green.xml`과 동일 구조, 색상만 `#D97706`

## 수정 파일 (2개)

### 3. `LogsViewModel.kt` 수정

- `app/src/main/java/com/qguarder/android/ui/logs/LogsViewModel.kt`

- `_stats: MutableStateFlow<ScanSummaryDto?>` 추가 → 통계 카드용

- `_error: MutableSharedFlow<String>` 추가 → 에러 피드백

- `loadLogs()` 성공 시 `_stats.value = body?.summary` 세팅

- catch 블록에 에러 메시지 emit

### 4. `LogsFragment.kt` 수정

- `app/src/main/java/com/qguarder/android/ui/logs/LogsFragment.kt`

- `logsAdapter` lazy 초기화 + RecyclerView 연결

- `setupObserver()`에서:

- logs collect → `logsAdapter.submitList(list)` + empty state 토글

- stats collect → `tv_safe_percent`, `tv_stats_summary`, `tv_stats_detail` 바인딩

- error collect → Toast 표시

## 참고할 기존 코드

- `FavoritesAdapter.kt` — ListAdapter + DiffUtil 패턴

- `item_favorite.xml` — MaterialCardView 아이템 레이아웃

- `bg_badge_green.xml` — 뱃지 drawable 구조

- `ScanLogItemDto`, `ScanSummaryDto` — DTO (Dtos.kt:86-101)

## 검증

- `./gradlew assembleDebug` 빌드 확인

- 앱 실행 → 하단 "보안 로그" 탭 → empty state 표시 확인

- (서버 연동 후) 스캔 → 로그 탭에서 기록 + 통계 표시 확인