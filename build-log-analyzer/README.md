# 빌드 로그 분석 대시보드 — 기획서

## 1. 배경 및 문제 정의

프로젝트는 Jenkins로 여러 빌드 파이프라인(`Build_PS5`, `Build_Package`, `CookValidation_PS5`, `Analyze` 등)을 운영하고 있고, 빌드 실패가 발생하면 담당자가 콘솔 로그를 직접 열어 원인을 찾아야 했다. 이 과정에서 반복적으로 겪던 문제:

- 로그 한 건이 수십 MB에 달해 사람이 직접 읽고 원인을 특정하기 번거로움
- 같은 유형의 실패(P4 Librarian 캐시 누락, UBA 컴파일 오류 등)가 반복되는데도 이력이 남지 않아 "이게 몇 번째인지" 감으로만 파악
- 실패 원인이 특정 빌드 머신에 몰려있는지, 특정 팀 담당 코드에서 자주 나는지 등 통계적 파악이 어려움

이 도구는 **"빌드 실패 로그를 Claude로 분석 → 구조화된 결과를 누적 기록 → 통계로 원인 패턴을 파악"** 하는 흐름을 웹툴로 지원하기 위해 만들어졌다.

## 2. 목적

- 빌드 실패의 **근본 원인 카테고리 통계**를 파악한다 (컴파일/링크/쿡/에셋/인프라 에러 중 무엇이 가장 많은지)
- 실패가 **특정 팀, 특정 빌드 머신, 특정 파이프라인 단계**에 편중되는지 파악한다
- 반복되는 에러 패턴(TOP 5)을 식별해 재발 방지 논의의 근거로 쓴다
- 실패 이력과 조치 내용을 검색 가능한 형태로 남겨, "이거 저번에도 있었는데 어떻게 해결했지?" 를 바로 찾을 수 있게 한다

### 비범위 (하지 않는 것)

- **빌드 성공 이력은 추적하지 않는다.** 목적이 "실패 원인 통계"이지 전체 빌드 상태 모니터링이 아니기 때문에, 성공 건까지 남기면 분석할 내용 없는 레코드만 쌓여 실패 패턴을 오히려 흐린다. 실패가 나중에 해결되면 원래 실패 레코드의 `resolved`/`action`/`memo`를 갱신하는 방식으로 "해결됨"을 표현한다 (별도 SUCCESS 레코드를 만들지 않음).
- **자동 삭제/정리 기능은 없다.** 이력은 쌓이기만 하며, 필요 시 수동으로 `data/records.json`을 편집한다.
- **알림/모니터링(예: 슬랙 알림) 기능은 없다.** 어디까지나 "분석 결과를 기록하고 통계를 보여주는" 도구다.

## 3. 사용 흐름

### 3-1. 기본 흐름 (수동)

```
1. Jenkins 빌드 실패 발생
2. 담당자가 콘솔 로그를 다운로드해서 Claude 채팅에 파일 경로 전달
3. Claude가 로그를 읽고 분석 → 정해진 JSON 스키마로 결과 산출
4. 웹툴 "분석 결과 입력" 탭에서 JSON을 붙여넣거나, Claude가 API로 직접 저장
5. 이력 목록 / 대시보드에 즉시 반영
```

### 3-2. Jenkins API 연동 흐름 (온디맨드)

빌드머신에서 로그 파일을 수동으로 다운로드하지 않아도, Jenkins REST API 인증 정보(`.env`)가 설정되어 있으면 "OO 잡 #번호 분석해줘"라고 요청하는 것만으로 Claude가 직접 Jenkins에서 콘솔 로그를 받아와 동일하게 분석한다.

- **채택하지 않은 대안 — 스케줄러 자동 폴링**: Jenkins 잡들을 주기적으로 폴링해서 실패를 자동 감지하는 방식도 검토했으나, (1) Claude Code 앱이 켜져 있어야만 동작하는 제약이 있고 (2) 사람이 로그를 한 번도 보지 않고 바로 기록되어 오탐 가능성이 있어, 현재는 **사람이 트리거하는 온디맨드 분석**으로 운영하기로 결정했다.
- 인증 토큰은 채팅에 붙여넣지 않고 `.env` 파일에 직접 저장하는 방식을 사용한다 (대화 로그에 credential이 남는 것을 방지).

## 4. 타겟 사용자

- 빌드 파이프라인을 관리하는 PM/엔지니어 (본 프로젝트 기준 1인 운영)
- 필요 시 엔진팀/프로그램팀 등 담당팀에 "이 문제 몇 번째 반복이다" 근거를 전달하는 용도로도 활용

## 5. 화면 구성 (3개 탭)

화면은 다음 순서로 배치되어 있다 (좌측부터):

1. **대시보드** — 기본 진입 화면. 요약 카드 + 차트 6종으로 현재 상태를 한눈에 파악
2. **이력 목록** — 필터/검색 가능한 전체 실패 이력 테이블. 행 클릭 시 상세/수정
3. **분석 결과 입력** — Claude 분석 결과 JSON을 붙여넣어 폼에 자동 반영 후 저장

탭 순서와 기본 진입 화면은 "실패가 났을 때 입력하는 것"보다 "평소 상태를 확인하는 것"이 더 빈번한 사용 패턴이라는 판단 하에 대시보드를 맨 앞/기본으로 배치했다.

## 6. 핵심 설계 원칙

- **확실하지 않으면 null로 남긴다.** 원인 CL/작업자를 추측만으로 채우지 않고, 근거가 부족하면 `memo`에 불확실성을 명시한다. (예: 동기화된 최신 CL이 있어도 그게 실제 원인인지 확신 없으면 `cl_number: null` + memo에 참고용으로만 기록)
- **같은 빌드번호라도 잡(job)이 다르면 다른 빌드다.** Jenkins 빌드 번호는 잡별로 독립적으로 매겨지므로, `job_name` + `build_number` 조합이 실제 식별자다.
- **원인이 하드웨어/인프라 성격인지 코드 성격인지 구분해서 기록한다.** 같은 빌드 머신에서 서로 무관해 보이는 크래시가 반복되면 그 자체가 하드웨어 결함의 신호일 수 있다는 점을 메모에 남겨, 다음 담당자가 "또 이 머신이네" 하고 넘기지 않도록 한다.
- **자동 저장은 신중하게.** 로그를 읽고 분석하는 모든 순간이 "기록해달라"는 뜻은 아니다 — 단순 호기심(예: "이 빌드머신이 뭐하는 거야?")으로 로그를 확인한 경우에는 저장 여부를 먼저 확인한다.

## 7. 향후 고려 사항 (미확정)

- 사내망에서 다른 PC로 접근 가능한지 (방화벽 정책에 따라 다름, 미검증)
- 여러 명이 동시에 쓸 때의 동시성 (현재 서버 내부 쓰기 큐로 단순 직렬화만 되어 있음, 실사용 트래픽 미검증)
- 실패율(시도 대비 실패 비율)이 필요해지면 Jenkins API로 총 시도 횟수만 가볍게 가져오는 방식 검토 (전체 성공 빌드를 하나하나 분석해 기록하지는 않음)

# 빌드 로그 분석 대시보드 — 상세 기능서

## 1. 기술 스택 / 실행

- **런타임**: Node.js + Express (`server.js`)
- **저장소**: JSON 파일 (`data/records.json`), 별도 DB 없음
- **프론트엔드**: 순수 HTML/CSS/JS (프레임워크 없음), 차트는 Chart.js를 로컬 vendoring(`public/vendor/chart.umd.js`)
- **포트**: 기본 `9100`, 환경변수 `PORT`로 변경 가능. `0.0.0.0` 바인딩으로 사내망 접근 허용
- **실행**: `npm install` → `npm start`

```
build-log-dashboard/
├── server.js                 # Express 서버 (API + 정적 파일 서빙)
├── .env                       # Jenkins API 인증 정보 (git 미포함)
├── data/records.json          # 실제 데이터 저장소
└── public/
    ├── index.html
    ├── css/style.css
    ├── js/app.js               # 탭 전환, 폼, 필터, 차트 로직
    └── vendor/chart.umd.js
```

## 2. 데이터 모델

레코드 1건 = 빌드 실패 분석 결과 1건. `data/records.json`에 배열로 저장.

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `id` | string (UUID) | 자동생성 | 서버가 `crypto.randomUUID()`로 생성, 클라이언트가 지정 불가 |
| `created_at` | ISO datetime | 자동생성 | 최초 생성 시각, 수정 시에도 불변 |
| `updated_at` | ISO datetime | 자동생성 | PUT/PATCH 시마다 갱신 |
| `build_number` | number | **필수** | Jenkins 빌드 번호. `job_name`과 조합해야 유일 식별자 |
| `job_name` | string | **필수** | Jenkins 잡 이름 (예: `Build_PS5`) |
| `build_node` | string \| null | 선택 | 실제 빌드가 실행된 Jenkins 노드/머신 (예: `Dev-Build`, `BUILD-04`) |
| `date` | string(YYYY-MM-DD) \| null | 선택 | 빌드 실행 날짜. 모르면 null (월별 트렌드 집계에서 제외됨) |
| `result` | `"FAILURE"` \| `"SUCCESS"` | 기본 FAILURE | 현재 UI/워크플로우는 FAILURE 위주 (§기획서 비범위 참고) |
| `failed_stage` | string \| null | 선택 | Jenkins 파이프라인 세부 스테이지명 (예: `Update Client`, `Compile Editor`). **`job_name`과는 다른 개념** — job은 어떤 파이프라인인지, stage는 그 안의 어느 단계인지 |
| `error_type` | string \| null | 선택 | 에러 유형 한 줄 요약 |
| `error_category` | 고정값 5종 \| null | 선택 | 아래 §3 참고 |
| `responsible_team` | 고정값 9종 \| null | 선택 | 아래 §3 참고 |
| `author` | string \| null | 선택 | 원인 제공 작업자 P4 계정. **확신 없으면 반드시 null** |
| `cl_number` | number \| null | 선택 | 원인 changelist 번호. **확신 없으면 반드시 null**, 참고용 CL은 memo에 기록 |
| `affected_file` | string \| null | 선택 | 문제가 된 파일/에셋 경로 |
| `error_log` | string | 선택 (기본 `''`) | 에러 로그 핵심 원문 (verbatim) |
| `summary` | string | 선택 (기본 `''`) | 원인 한 줄 요약 (한국어) |
| `action` | string | 선택 (기본 `''`) | 조치 방법/진행상황 |
| `resolved` | boolean | 기본 `false` | 해결 여부 |
| `memo` | string | 선택 (기본 `''`) | 불확실성, 참고 사항, 진행상황 로그 등 자유 기술 |

### 고정값 목록

```js
error_category: ['컴파일 에러', '링크 에러', '쿡 에러', '에셋 에러', '인프라 에러']
responsible_team: ['프로그램팀', '아티스트', '엔진팀', '사운드팀', '애니메이션팀',
                    '연출팀', '캐릭터팀', 'TA팀', 'FX팀']
```

`failed_stage`, `build_node`, `job_name`은 고정값이 아니라 **기본 목록 + 실제 데이터에서 발견된 값**을 합쳐 자동완성(datalist)으로 제공한다 (서버 `server.js`의 `DEFAULT_FAILED_STAGES` / `DEFAULT_BUILD_NODES` / `DEFAULT_JOB_NAMES` 상수 + `/api/meta`에서 기존 레코드 스캔하여 합집합).

- `DEFAULT_BUILD_NODES`: `Built-In Node`, `Analyze`, `DailyBuilder`, `Dev-Build`, `Dev-Build-02`, `Editor`, `Engine`, `BUILD-04` (Jenkins 빌드 실행 상태 패널 기준)
- `DEFAULT_JOB_NAMES`: `Build_PS5`, `Build_Package`, `CookValidation_PS5`, `Analyze`
- `DEFAULT_FAILED_STAGES`: `Build Editor Module`, `Build Game Module`, `Cook Content`, `Update Client`

## 3. API 명세

베이스: `http://localhost:9100` (또는 `PORT` 환경변수로 설정된 포트)

### `GET /api/meta`
고정값 + 자동완성 후보 목록 반환.
```json
{
  "errorCategories": [...5종],
  "responsibleTeams": [...9종],
  "failedStages": [...기본4종 + 발견된 값],
  "buildNodes": [...기본8종 + 발견된 값],
  "jobNames": [...기본4종 + 발견된 값]
}
```

### `GET /api/records`
목록 조회. 쿼리 파라미터(전부 선택, AND 조건):

| 파라미터 | 설명 |
|---|---|
| `from`, `to` | `date` 범위 필터 (YYYY-MM-DD, 문자열 비교) |
| `category` | `error_category` 정확히 일치 |
| `team` | `responsible_team` 정확히 일치 |
| `node` | `build_node` 정확히 일치 |
| `job` | `job_name` 정확히 일치 |
| `resolved` | `"true"` / `"false"` 문자열 |
| `q` | 키워드 검색 — `affected_file`, `author`, `cl_number`, `job_name`, `error_type`, `summary`를 합쳐 대소문자 무시 부분일치 |

정렬: `date` 내림차순 → 동일하면 `build_number` 내림차순.
응답: `{ records: [...], total: N }`

### `GET /api/records/:id`
단건 조회. 없으면 404.

### `POST /api/records`
신규 생성. `job_name`, `build_number` 없으면 400. `id`/`created_at`/`updated_at`은 서버가 채움. 나머지 필드는 body에 없으면 §2의 기본값(대부분 `null` 또는 `''`)으로 채워짐 — 즉 **부분 필드만 보내도 되지만, 안 보낸 필드는 새로 만들어지는 값이 아니라 무조건 기본값으로 초기화됨** (PUT과의 차이 주의).

### `PUT /api/records/:id`
전체 교체. `id`/`created_at`은 기존 값 유지, 나머지는 `POST`와 동일한 정규화 로직(`normalizeRecord`)을 거치되 body에 없는 필드는 **기존 레코드 값을 유지**(기본값으로 초기화되지 않음 — `existing` 인자를 넘기기 때문). 즉 "폼 전체를 다시 제출"하는 시나리오(상세 모달 수정 저장)에 사용.

### `PATCH /api/records/:id`
부분 수정. body에 있는 키만 얕은 병합(`{...existing, ...body}`). `id`/`created_at`은 덮어쓰기 불가. 이력 목록에서 해결여부만 토글하거나, 진행상황 메모만 추가할 때 사용.

### `GET /api/stats`
대시보드 집계. `result !== 'SUCCESS'`인 레코드만 `failures`로 집계 대상 삼음 (현재는 사실상 전부 FAILURE).

```json
{
  "summary": {
    "totalCount": 전체 레코드 수 (SUCCESS 포함),
    "thisMonthFailureCount": 이번 달(date 기준) FAILURE 건수,
    "unresolvedCount": resolved=false인 FAILURE 건수,
    "topCategory": categoryCounts 1위 라벨 또는 null,
    "topNode": nodeCounts 1위 라벨 또는 null
  },
  "categoryCounts": [{label, count}, ...]   // error_category별, 내림차순
  "teamResolvedCounts": [...]                // resolved=true인 것만 responsible_team별 집계
  "monthlyTrend": [{month:"YYYY-MM", count}] // date 있는 것만, 오름차순
  "topErrorTypes": [...]                     // error_type별 상위 5개
  "stageCounts": [...]                       // failed_stage별
  "nodeCounts": [...]                        // build_node별
}
```

**주의**: `label`이 `null`/`undefined`/`''`인 항목은 집계에서 제외됨 (`countBy` 함수 내 `if (key === null...) return`).

레코드 **삭제 API는 없음**. 정리가 필요하면 `data/records.json`을 직접 편집.

## 4. 화면별 기능

### 4-1. 대시보드 탭 (기본 진입 화면)

**요약 카드 5종** (`renderSummaryCards`):
1. 전체 분석 건수
2. 이번 달 실패 건수
3. 미해결 건수
4. 가장 많은 에러 카테고리
5. 문제 최다 빌드머신

**차트 6종** (배치 순서, 위→아래):
1. **에러 카테고리별 빈도** (도넛) — `categoryCounts`. 마우스 오버 시 툴팁에 `건수 (퍼센트%)` 함께 표시 (전체 대비 비율, 소수 첫째자리)
2. **담당팀별 해결 빈도** (막대) — `teamResolvedCounts`. **주의**: 이 차트만 유일하게 "해결된 것만" 집계함 (팀이 실제로 마무리한 건수를 보기 위함, 미해결 포함 전체 건수가 아님)
3. **반복 에러 TOP 5** (가로 막대) — `topErrorTypes`, 상위 5개만
4. **실패 스테이지별 빈도** (막대) — `stageCounts`
5. **빌드머신별 실패 빈도** (가로 막대) — `nodeCounts`
6. **월별 빌드 실패 트렌드** (라인, 2칸 너비) — `monthlyTrend`. 데이터가 쌓여야 의미가 생기는 지표라 의도적으로 맨 아래 배치

차트 로딩 타이밍: 최초 페이지 로드 시 `window.load` 이벤트 이후에 그림 (스타일시트 적용 전에 그리면 Chart.js가 캔버스를 기본 크기 300×150으로 고정해버리는 버그 회피). 대시보드 탭을 다시 클릭할 때마다 서버에서 최신 데이터로 재조회/재렌더링.

### 4-2. 이력 목록 탭

**필터 (전부 AND 조합, 값 바뀌면 즉시 재조회)**:
- 기간 (From/To 날짜)
- 잡 이름
- 에러 카테고리
- 담당팀
- 빌드머신
- 해결 여부 (전체/미해결/해결)
- 키워드 검색 (입력 300ms debounce)
- "필터 초기화" 버튼으로 전체 리셋

**테이블 컬럼**: 날짜, 빌드번호, 잡이름, 빌드머신, 실패스테이지, 에러카테고리, 담당팀, 작업자, 해결여부

**행 동작**:
- 해결여부 배지를 클릭하면 그 자리에서 즉시 토글 (모달 열지 않고 `PATCH`로 반영, `event.stopPropagation()`으로 행 클릭과 분리)
- 배지 외 영역 클릭 시 상세 모달이 열리고, 해당 레코드의 전체 필드를 폼에 채워서 보여줌

### 4-3. 상세보기/수정 모달

이력 목록에서 행 클릭 시 오픈. 전체 필드를 편집 가능한 폼으로 표시하고, "수정 저장" 클릭 시 `PUT /api/records/:id` 호출 후 이력 목록 재조회 + 0.5초 후 모달 자동 닫힘.

### 4-4. 분석 결과 입력 탭

1. JSON 붙여넣기 텍스트박스 + "파싱해서 폼에 채우기" 버튼 — `JSON.parse` 실패 시 에러 메시지 표시, 성공 시 폼에 자동 반영
2. 자동 반영된 폼은 그대로 수정 가능
3. "저장" 클릭 시 `POST /api/records` 호출. `job_name`/`build_number` 미입력 시 클라이언트단에서 먼저 검증 후 막음
4. 저장 성공 시 폼과 붙여넣기 textarea 초기화, `result: FAILURE, resolved: false`로 리셋

### 4-5. 폼 필드 렌더링 공통 로직

`FIELDS` 배열(§2 데이터 모델과 1:1 대응)을 기준으로 입력 탭 폼과 상세 모달 폼을 **동일한 코드로 동적 생성**한다 (`buildForm`). 필드 타입:

- `text`/`number`/`date`: 기본 input
- `select` + `metaKey`: `/api/meta` 응답의 해당 배열로 옵션 채움 (예: `error_category` → `errorCategories`)
- `combo` + `metaKey`: 자유 입력 + datalist 자동완성 (예: `build_node` → `buildNodes`). select와 달리 목록에 없는 값도 직접 입력 가능
- `textarea` (`full: true`인 경우 폼 전체 너비 차지)
- `checkbox`: `resolved`

## 5. 검증 규칙

- **서버**: `job_name` 존재 + `build_number`가 undefined/null/빈문자열이 아닐 것 (POST/PUT 공통, `validateRecord`)
- **클라이언트(입력 탭)**: 저장 버튼 클릭 시 위와 동일한 조건을 한 번 더 확인 후 API 호출 (사용자 피드백을 서버 왕복 없이 즉시 주기 위함)
- 그 외 필드는 전부 선택 — 특히 `author`/`cl_number`는 **확신 없으면 비워두는 것이 원칙** (기획서 §6 참고)

## 6. Jenkins API 연동 (온디맨드 분석용, 대시보드 서버와는 별개)

대시보드 서버 자체에는 Jenkins 연동 API가 없다 — 이는 Claude가 대화 중 필요 시 직접 호출하는 방식으로, `D:\PM\build-log-dashboard\.env`에 아래 값을 저장해두고 사용한다.

```
JENKINS_URL=http://builder:8080
JENKINS_USER=<계정>
JENKINS_API_TOKEN=<API 토큰>
```

- 인증: HTTP Basic (`user:token`)
- 잡 상태 조회: `GET {JENKINS_URL}/job/{job}/{build}/api/json`
- 콘솔 로그: `GET {JENKINS_URL}/job/{job}/{build}/consoleText`
- `.env`는 `.gitignore`에 포함되어 있고 `public/` 바깥에 위치해 웹으로 노출되지 않음
- **verbose 로깅(curl -v 등) 사용 금지** — Basic Auth 헤더가 그대로 노출되며 base64 디코드 시 토큰 원문이 드러남

## 7. 알려진 제약사항

- 레코드 삭제 API 없음
- 동시 쓰기는 in-memory 큐(`queueWrite`)로 단순 직렬화만 되어 있고, 다중 사용자 동시 접속 시나리오는 실사용 트래픽으로 검증되지 않음
- `build_node`/`job_name`이 없는 과거 시드 데이터는 필터/차트에서 `-` 또는 집계 제외로 처리됨
- 인증/권한 체계 없음 (사내망 접근 가능자는 누구나 CRUD 가능)
