---
title: apidesc
---
![[Pasted image 20260315200406.png]]

![[api_spec_v1.html]]

# 점수 산정 로직
vt 점수 환산
```python
vt_raw = (detected / total) * 100
ml_raw = ml_prob * 100

if vt_detected >= 1: vt_weight, ml_weight = 0.8, 0.2
else: vt_weight, ml_weight = 0.2, 0.8

final score = (vt_raw * vt_weight) + (ml_raw * ml_weight)

```

# 런타임 & 프레임워크
python 3.11.x # 람다
fastapi 0.111 # mangum 0.17 : asgi->lambda 어댑터
pydantic v2 # Score breakdown, XAIFeature 모델 정의
python-jose 3.3 # JWT HS256 / bcrypt passlib 1.7
LightGBM 4.x # ml 모델, joblib 직렬화 -> s3 저장
SHAP 0.45 # xai : feature contribution 계산 -> xai_features JSON 저장
API v3 # 바토: httpx async 호출, detected/total -> vt_raw_score
tldextract 5.x # feature 추출: 도메인 파싱 urllib.parse + regex
postgresql 16.x # sqlalchemy 2.0 async + asyncpg, Alembic 1.13 migration
redis 7.2.x # ElastCache - scan:{url_hash} TTL 6시간임
Room / SQLite # 로컬디비


# S3 버킷 구조
```
phishing-detector-bucket/ 
│ 
├── models/ 
│ 
├── v1/ 
│ 
│ 
├── model.pkl # LightGBM joblib 직렬화 
│ 
│ 
├── scaler.pkl # StandardScaler 
│ 
│ 
├── feature_names.json # 피처 순서 (추론 시 필수) 
│ 
│ 
└── metadata.json # { accuracy, f1, trained_at, sample_count } 
│ 
├── v2/ # 재학습마다 버전 증가 
│ 
└── latest/ # 항상 최신 버전 복사본 (Lambda가 여기서 로드) 
│ 
├── model.pkl 
│ 
├── scaler.pkl 
│ 
└── feature_names.json 
│ 
├── datasets/ 
│ 
├── base/ # 초기 학습 원본 (불변) 
│ 
│ 
└── phishing_urls_v1.csv 
│ 
└── approved/ # VT 검증 통과한 피드백 데이터 
│ 
├── feedback_2025Q1.csv 
│ 
└── feedback_2025Q2.csv 
│ 
└── training-logs/ 
└── run_20250310_v2.json # { f1_before, f1_after, promoted: true }
```

# 전체 백엔드 아키텍처
![[Pasted image 20260315203355.png]]


```YAML
  

<style>

.api-wrap { font-size: 13px; padding: 1rem 0; }

.api-group { margin-bottom: 1.5rem; }

.api-group-title { font-size: 15px; font-weight: 500; color: var(--color-text-primary); margin: 0 0 8px; border-bottom: 0.5px solid var(--color-border-tertiary); padding-bottom: 6px; }

.api-row { padding: 8px 10px; border-radius: var(--border-radius-md); border: 0.5px solid var(--color-border-tertiary); margin-bottom: 6px; background: var(--color-background-primary); }

.api-top { display: flex; align-items: flex-start; gap: 10px; }

.method { font-size: 11px; font-weight: 500; padding: 2px 7px; border-radius: 4px; min-width: 46px; text-align: center; flex-shrink: 0; margin-top: 2px; }

.m-post { background: #E6F1FB; color: #185FA5; }

.m-get { background: #EAF3DE; color: #3B6D11; }

.m-delete { background: #FCEBEB; color: #A32D2D; }

@media (prefers-color-scheme: dark) {

.m-post { background: #042C53; color: #85B7EB; }

.m-get { background: #173404; color: #97C459; }

.m-delete { background: #501313; color: #F09595; }

}

.endpoint { font-family: var(--font-mono); font-size: 12px; color: var(--color-text-primary); font-weight: 500; flex: 1; min-width: 0; word-break: break-all; }

.ep-desc { font-size: 12px; color: var(--color-text-secondary); margin-top: 2px; }

.right-col { display: flex; flex-direction: column; align-items: flex-end; gap: 4px; flex-shrink: 0; }

.auth-badge { font-size: 10px; padding: 1px 5px; border-radius: 3px; background: var(--color-background-secondary); color: var(--color-text-secondary); border: 0.5px solid var(--color-border-tertiary); }

.schema-toggle { font-size: 11px; color: var(--color-text-info); cursor: pointer; background: none; border: none; padding: 0; }

.schema-box { display: none; background: var(--color-background-secondary); border-radius: 6px; padding: 10px 12px; margin-top: 8px; font-family: var(--font-mono); font-size: 11px; color: var(--color-text-secondary); white-space: pre; line-height: 1.7; border: 0.5px solid var(--color-border-tertiary); overflow-x: auto; }

.schema-box.open { display: block; }

.highlight { color: var(--color-text-primary); font-weight: 500; }

.note { font-size: 11px; color: var(--color-text-warning); background: var(--color-background-warning); border-radius: 4px; padding: 3px 8px; margin-top: 6px; display: inline-block; }

</style>

<div class="api-wrap">

  

<div class="api-group">

<div class="api-group-title">Auth — /api/v1/auth</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-post">POST</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/auth/register</div>

<div class="ep-desc">회원가입</div>

</div>

<div class="right-col"><button class="schema-toggle" onclick="tog('s1')">schema ▾</button></div>

</div>

<div class="schema-box" id="s1"><span class="highlight">Request</span>

{ email: string, password: string, nickname: string }

  

<span class="highlight">Response 201</span>

{ user_id: uuid, email, nickname, created_at }</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-post">POST</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/auth/login</div>

<div class="ep-desc">로그인 — access token 반환, refresh token HttpOnly 쿠키 설정</div>

</div>

<div class="right-col"><button class="schema-toggle" onclick="tog('s2')">schema ▾</button></div>

</div>

<div class="schema-box" id="s2"><span class="highlight">Request</span>

{ email: string, password: string }

  

<span class="highlight">Response 200</span>

{ access_token: string, token_type: "bearer" }

Set-Cookie: refresh_token=...; HttpOnly; Secure; SameSite=Strict; Path=/api/v1/auth/refresh</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-post">POST</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/auth/refresh</div>

<div class="ep-desc">access token 재발급</div>

</div>

<div class="right-col"><button class="schema-toggle" onclick="tog('s3')">schema ▾</button></div>

</div>

<div class="schema-box" id="s3"><span class="highlight">Request</span>

Cookie: refresh_token=... (HttpOnly, 자동 전송)

  

<span class="highlight">Response 200</span>

{ access_token: string, token_type: "bearer" }</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-post">POST</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/auth/logout</div>

<div class="ep-desc">로그아웃 — 서버 refresh token 무효화 + 쿠키 삭제</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s4')">schema ▾</button></div>

</div>

<div class="schema-box" id="s4"><span class="highlight">Request</span>

Header: Authorization: Bearer {access_token}

  

<span class="highlight">Response 200</span>

{ message: "logged out" }

Set-Cookie: refresh_token=; Max-Age=0</div>

</div>

</div>

  

<div class="api-group">

<div class="api-group-title">Scan — /api/v1/scan ★ 핵심 변경</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-post">POST</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/scan/url</div>

<div class="ep-desc">URL 위험도 분석 — VirusTotal + LightGBM Dynamic Weighting 판단</div>

<div class="note">모든 URL에 VT + ML 이중 점수 산정</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s5')">schema ▾</button></div>

</div>

<div class="schema-box" id="s5"><span class="highlight">Request</span>

{ url: string, source: "camera" | "manual" }

  

<span class="highlight">Response 200</span>

{

scan_id: uuid,

url: string,

url_hash: string,

cached: boolean,

  

<span class="highlight">// 최종 판단</span>

risk_level: "safe" | "suspicious" | "dangerous",

final_score: 72.4, // 0~100

weighting_case: "A" | "B",

threshold_message: "큐싱 사이트로 확정됩니다. 접속이 차단되었습니다.",

  

<span class="highlight">// 점수 상세 (Case A: VT탐지 1건 이상)</span>

score_breakdown: {

weighting_case: "A",

vt: {

detected_engines: 5,

total_engines: 90,

raw_score: 55.6, // 5/90 * 100

weight: 0.8,

weighted_score: 44.4 // raw_score * weight

},

ml: {

probability: 0.351,

raw_score: 35.1, // probability * 100

weight: 0.2,

weighted_score: 7.0 // raw_score * weight

},

final_score: 51.4 // vt.weighted + ml.weighted

},

  

<span class="highlight">// XAI (ML 기여도 상위 피처)</span>

xai_features: [

{ feature: "url_length", value: 142, contribution: 0.21, direction: "danger" },

{ feature: "has_ip_address", value: 1, contribution: 0.35, direction: "danger" },

{ feature: "special_char_count",value: 7, contribution: 0.18, direction: "danger" },

{ feature: "domain_age_days", value: 3, contribution: 0.12, direction: "danger" }

],

  

scanned_at: "2025-03-15T12:00:00Z"

}</div>

</div>

</div>

  

<div class="api-group">

<div class="api-group-title">Report — /api/v1/report</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-post">POST</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/report</div>

<div class="ep-desc">URL 신고 — feedback_queue 적재 (VT 교차검증은 별도 비동기 처리)</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s6')">schema ▾</button></div>

</div>

<div class="schema-box" id="s6"><span class="highlight">Request</span>

{ scan_log_id: uuid, url: string, reason?: string }

  

<span class="highlight">Response 202</span>

{ report_id: uuid, status: "queued", message: "검토 후 블랙리스트에 반영됩니다." }</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-get">GET</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/report/blacklist?page=1&size=20</div>

<div class="ep-desc">블랙리스트 랭킹 — 신고 횟수 + 최고 위험 점수 기준 정렬</div>

</div>

<div class="right-col"><button class="schema-toggle" onclick="tog('s7')">schema ▾</button></div>

</div>

<div class="schema-box" id="s7"><span class="highlight">Response 200</span>

{

items: [

{

rank: 1,

url: string,

report_count: 42,

risk_level: "dangerous",

final_score: 91.2,

weighting_case: "A",

last_reported_at: string

}, ...

],

total: 200, page: 1, size: 20

}</div>

</div>

</div>

  

<div class="api-group">

<div class="api-group-title">User — /api/v1/users/me</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-get">GET</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/users/me/scan-logs?from=2025-01&to=2025-03</div>

<div class="ep-desc">내 스캔 이력 — 캘린더 집계 + 점수 분포 포함</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s8')">schema ▾</button></div>

</div>

<div class="schema-box" id="s8"><span class="highlight">Response 200</span>

{

summary: {

total_scans: 87,

safe_count: 61, suspicious_count: 18, dangerous_count: 8,

case_a_count: 12, case_b_count: 75

},

calendar: {

"2025-03-10": { count: 3, dangerous: 1, avg_score: 45.2 },

...

},

logs: [

{ scan_id, url, risk_level, final_score, weighting_case, scanned_at }, ...

]

}</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-get">GET</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/users/me/scan-logs/{scan_id}/detail</div>

<div class="ep-desc">특정 스캔 상세 — score_breakdown + xai_features 전체 조회</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s9')">schema ▾</button></div>

</div>

<div class="schema-box" id="s9"><span class="highlight">Response 200</span>

{

scan_id, url, risk_level, final_score, weighting_case,

score_breakdown: { vt: {...}, ml: {...}, final_score },

xai_features: [ { feature, value, contribution, direction }, ... ],

scanned_at

}</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-get">GET</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/users/me/bookmarks</div>

<div class="ep-desc">즐겨찾기 목록</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s10')">schema ▾</button></div>

</div>

<div class="schema-box" id="s10"><span class="highlight">Response 200</span>

{ items: [ { bookmark_id, url, title, risk_level, final_score, created_at } ] }</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-post">POST</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/users/me/bookmarks</div>

<div class="ep-desc">즐겨찾기 추가</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s11')">schema ▾</button></div>

</div>

<div class="schema-box" id="s11"><span class="highlight">Request</span>

{ url: string, title?: string, risk_level: string, final_score: float }

  

<span class="highlight">Response 201</span>

{ bookmark_id: uuid, created_at }</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-delete">DEL</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/users/me/bookmarks/{bookmark_id}</div>

<div class="ep-desc">즐겨찾기 삭제</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s12')">schema ▾</button></div>

</div>

<div class="schema-box" id="s12"><span class="highlight">Response 200</span>

{ message: "deleted" }</div>

</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-get">GET</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/api/v1/users/me/visits?page=1&size=20</div>

<div class="ep-desc">방문 기록</div>

</div>

<div class="right-col"><span class="auth-badge">JWT</span><button class="schema-toggle" onclick="tog('s13')">schema ▾</button></div>

</div>

<div class="schema-box" id="s13"><span class="highlight">Response 200</span>

{ items: [ { url, risk_level, final_score, visited_at } ], total }</div>

</div>

</div>

  

<div class="api-group">

<div class="api-group-title">Internal / MLOps (서버 내부 전용)</div>

  

<div class="api-row">

<div class="api-top">

<span class="method m-post">POST</span>

<div style="flex:1;min-width:0">

<div class="endpoint">/internal/model/reload</div>

<div class="ep-desc">재학습 완료 후 Lambda Training이 호출 → 최신 모델 Hot Reload</div>

</div>

<div class="right-col"><span class="auth-badge">Internal</span><button class="schema-toggle" onclick="tog('s14')">schema ▾</button></div>

</div>

<div class="schema-box" id="s14"><span class="highlight">Request</span>

Header: X-Internal-Token: {secret}

{ model_version: "v12", s3_key: "models/v12/model.pkl" }

  

<span class="highlight">Response 200</span>

{ loaded_version: "v12", loaded_at }</div>

</div>

</div>

  

</div>

<script>

function tog(id) {

const el = document.getElementById(id);

el.classList.toggle('open');

}

</script>
```


