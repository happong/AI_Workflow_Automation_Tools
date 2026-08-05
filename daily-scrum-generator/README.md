# 데일리 스크럼 자동 생성 - 컨텐츠 기획서

## 1. 개요

### 배경
스페이스의 데일리 스크럼 Confluence 페이지를 매일 수기로 생성하던 것을 자동화. 운영 중 "어제 한 일" 컬럼에 하루 전이 아닌 이틀 전 데이터가 들어가는 버그가 제보되어, 원인 분석 및 수정과 함께 운영 환경(빌드머신) 이전을 진행함.

### 목적
- 매 영업일 아침, 전날 페이지를 기반으로 당일 데일리 스크럼 페이지를 자동 생성
- 팀원들이 별도로 "어제 한 일" 칸을 직접 옮겨 적는 수작업을 제거
- 자동화 실패/이상 상황을 담당자가 즉시 인지할 수 있도록 알림 체계 마련

## 2. 동작 요구사항

### 2.1 생성 시점
- 매일 **오전 7:00**에 실행되어 **당일** 데일리 스크럼 페이지를 생성
- 페이지 제목 형식: `YY.MM.DD 데일리 스크럼` (예: `26.07.15 데일리 스크럼`)

### 2.2 생성 제외 조건
- **토요일/일요일**: 생성하지 않음
- **한국 공휴일**: 생성하지 않음 (`holidays` 라이브러리의 KR 공휴일 기준)

### 2.3 페이지 생성 규칙
전날(직전 영업일) 페이지를 기준으로 당일 페이지를 생성하며, 표의 각 행에 대해 아래 규칙을 적용한다.

| 컬럼 | 규칙 |
|---|---|
| 어제 한 일 | 전날 페이지의 **"오늘 할일"** 내용을 그대로 복사 (텍스트, 이미지, 동영상 첨부 포함) |
| 오늘 할일 | 빈 값으로 초기화 |
| 공유 필요 사항 | 전날 페이지의 내용을 그대로 유지 (변경하지 않음) |

- 이름/파트/팀 등 나머지 컬럼 및 표 구조는 전날 페이지 그대로 유지
- 첨부된 이미지/동영상은 원본이 실제로 업로드된 페이지를 계속 참조하는 방식으로 정상 표시됨

### 2.4 페이지 정렬
- 생성된 페이지는 상위 페이지 하위 목록 맨 위로 자동 이동

### 2.5 중복 생성 방지
- 이미 동일 제목의 페이지가 존재하면 생성하지 않고 스킵 (운영 모드 한정)

## 3. 알림 정책

| 상황 | 알림 채널 | 대상 |
|---|---|---|
| 페이지 생성 완료 | Teams 팀 채널 | 팀 전체 |
| 스크립트 실행 중 오류 발생(전날 페이지 못 찾음, API 실패 등) | Teams 팀 채널 | 팀 전체 |
| 표의 특정 행이 예상치 못한 구조(컬럼 개수 이상)라 자동 복사가 스킵된 경우 | 개인 Teams 채널 | 자동화 담당자만 |

- "표 구조 이상" 알림을 팀 채널이 아닌 개인 채널로 분리한 이유: 팀원 입장에서 매번 노출될 필요 없는 운영/유지보수성 정보이기 때문

## 4. 알려진 제약 / 향후 고려사항

- 표의 행 구조가 비정상(컬럼 개수가 4/5/6이 아닌 경우)이면 해당 행은 자동 복사 없이 그대로 유지되며, 개인 알림으로만 확인 가능 (자동 보정은 하지 않음 — 잘못 보정했을 때의 위험이 더 크다고 판단)
- 실행 시각을 07:00로 정한 이유: 18:00 실행 시 당일 늦게까지 편집하는 인원의 "오늘 할일" 내용이 스냅샷 시점 이후로 반영되어 다음날 페이지에 누락되는 문제가 있었음

#  데일리 스크럼 자동 생성 - 상세 기술서

## 1. 시스템 구성

```
DailyScrum/
├── create_daily_scrum.py   # 핵심 로직 (페이지 조회/생성, 알림)
├── Check_Storage.py        # 특정 페이지의 표 구조(td 개수) 점검용 디버그 스크립트
├── run_daily_scrum.bat     # 실행 진입점 (환경변수 설정 + 스크립트 실행)
├── register_task.ps1       # Windows Task Scheduler 등록 스크립트
├── env.example.txt         # 환경변수 예시
└── logs/
    └── run.log             # 실행 요약 로그 (날짜/ERRORLEVEL)
```

### 배포 환경
- **실행 위치**: 빌드머신 (`C:\PM\_DailyScrum\_DailyScrum`)
- **Confluence 계정**: `manager@ncsoft.com` (서비스 계정, API 토큰 기반 인증)
- **Task Scheduler**: `DailyScrum` 태스크, SYSTEM 계정, 평일 07:00 트리거

## 2. 핵심 로직 (`create_daily_scrum.py`)

### 2.1 대상 날짜 계산
```python
if test_mode or today.hour >= 18:
    target_day = get_next_business_day(today)   # 다음 영업일 페이지 생성
else:
    target_day = today                          # 당일 페이지 생성
prev_day = get_prev_business_day(target_day)     # 참조할 전날 영업일
```
- 07:00 스케줄 기준으로는 항상 `target_day = today` 분기를 탐
- `--test` 플래그 사용 시에는 시간 무관하게 항상 다음 영업일 페이지를 생성 (실제 운영 페이지와 충돌 방지)

### 2.2 표 컬럼 판별 로직 (`build_new_page_storage`)
Confluence storage format의 `<tr>` 각각에 대해 `<td>` 개수(n)로 컬럼 위치를 판별한다. 팀/파트 컬럼이 `rowspan`으로 병합되어 있어 행마다 실제 `<td>` 개수가 다르기 때문.

| td 개수 | 어제 한일 idx | 오늘 할일 idx | 공유 필요사항 idx |
|---|---|---|---|
| 6 | 3 | 4 | 5 |
| 5 | 2 | 3 | 4 |
| 4 | 1 | 2 | 3 |
| 그 외 | - | - | - (자동 복사 스킵, 원본 그대로 유지 + 경고 기록) |

- 어제 한일 위치 ← 오늘 할일의 원본 내용을 복사
- 오늘 할일 위치 ← `<p local-id="empty"/>` 로 초기화
- 공유 필요사항 위치 ← 아무 처리 안 함 (원본 유지)
- 위 3가지 케이스에 해당하지 않는 행은 `warnings` 리스트에 기록되고, 원본 그대로 반환됨

### 2.3 첨부파일(이미지/동영상) 참조 처리 (`fix_attachment_refs`)
```python
if 'ri:page' in full:
    return full   # 이미 다른 페이지를 참조 중이면 그대로 유지 (원본이 실제로 업로드된 페이지 참조 보존)
if full.endswith('/>'):
    # self-closing 첨부파일 태그에 ri:page(참조 페이지) 를 새로 삽입
    ...
```
- Confluence는 첨부파일이 실제로 업로드된 페이지를 계속 참조하는 구조이므로, 이미 `ri:page`가 있는 태그는 건드리지 않는 것이 올바른 동작 (검증 완료 — 실운영 데이터로 확인)

### 2.4 로컬 ID 재생성
`clean_local_ids`: 모든 `local-id` 속성을 새 UUID로 치환하여, 원본 페이지와 새 페이지의 로컬 ID 충돌을 방지

### 2.5 알림
| 함수 | 트리거 | 대상 |
|---|---|---|
| `send_teams_notification` | 페이지 생성 성공 시 | `TEAMS_WEBHOOK_URL` (팀 채널) |
| `send_error_notification` | `main()`에서 처리되지 않은 예외 또는 `sys.exit(1)` 발생 시 | `TEAMS_WEBHOOK_URL` (팀 채널) |
| `send_personal_alert` | 표 구조 이상 행(`warnings`)이 하나 이상 발견된 경우 | `PERSONAL_WEBHOOK_URL` (개인 채널) |

`__main__` 블록에서 `main()`을 try/except로 감싸 정상 종료(`sys.exit(0)`, 영업일 아님/이미 존재)와 실제 에러(`sys.exit(1)`, 예외)를 구분해서, 에러일 때만 알림을 보냄.

## 3. 트러블슈팅 히스토리 (실제 발생 사례)

### 3.1 "어제 한 일"에 이틀 전 데이터가 들어가는 버그
- **원인**: 특정 행의 `<td>` 개수가 4/5/6이 아닌 이상 값(예: 7)이 되는 경우가 있었고, 이 경우 코드가 해당 행을 조용히 스킵하여 "오늘 할일"이 지워지지 않고, "어제 한일"도 갱신되지 않은 채 그대로 다음 페이지로 넘어감
- **검증 방법**: Confluence 버전 히스토리 API(`/rest/api/content/{id}?status=historical&version=N`)로 스크립트 실행 시점의 정확한 스냅샷을 재현하여, 실제 자동화 출력과 diff 비교
- **조치**: 이상 행 감지 시 경고를 수집해 개인 알림으로 전달 (자동 보정은 하지 않음)

### 3.2 빌드머신 이전 시 Confluence 403/404 (권한 문제)
- **증상**: 새 서비스 계정(`manager`)으로 특정 부모 페이지 밑에 페이지 생성 시 `404 NotFoundException: parent ID does not exist, or user does not have permissions`
- **진단**: 동일 계정으로 PROD 부모/스페이스 루트에는 생성이 성공하는 것을 확인 → 계정 전체 권한이 아닌 **특정 페이지 개별 제한(Restrictions)** 문제로 특정
- **조치**: 해당 페이지의 Restrictions에 `manager` 계정 추가로 해결

### 3.3 SYSTEM 계정(Task Scheduler) 실행 시 반환코드 1 (0x80070001)
- **증상**: 인터랙티브로 직접 실행하면 성공하는데, Task Scheduler(SYSTEM 계정)로 트리거하면 항상 `LastTaskResult=1`
- **1차 원인**: `run_daily_scrum.bat` 내부에서도 `logs\run.log`에 append 하는데, Task Scheduler 액션에서도 같은 파일에 외부 리다이렉션(`>> run.log 2>&1`)을 걸어 핸들 충돌 발생 → 로그 파일 분리로 일부 개선
- **2차 원인**: `2>&1` (핸들 복제 연산) 자체가 이 환경에서 실패(`could not duplicate handle`) → `2>&1` 대신 `2>별도파일`로 변경
- **최종 원인**: SYSTEM 계정은 콘솔이 없는 Session 0에서 실행되어, `cmd.exe /c ... >> file` 형태의 리다이렉션 자체가 근본적으로 실패. **cmd.exe 래퍼와 외부 리다이렉션을 완전히 제거하고 배치파일을 직접 실행**(`New-ScheduledTaskAction -Execute $BatFile`)하는 것으로 해결
- **경로 하드코딩 이슈**: `register_task.ps1`에 `$ScriptDir = "D:\DailyScrum"`가 하드코딩되어 있어 다른 머신에서 실행 시 드라이브를 찾을 수 없는 에러 발생 → `$ScriptDir = $PSScriptRoot`로 변경하여 스크립트 위치 기반으로 자동 해석되도록 수정

## 4. 환경변수

| 변수 | 설명 |
|---|---|
| `CONFLUENCE_BASE_URL` | Confluence 베이스 URL |
| `CONFLUENCE_USER_EMAIL` | API 인증 계정 이메일 |
| `CONFLUENCE_API_TOKEN` | API 토큰 (classic, 무제한 스코프) |
| `TEAMS_WEBHOOK_URL` | 팀 채널 알림용 웹훅 |
| `PERSONAL_WEBHOOK_URL` | 개인 알림용 웹훅 |

## 5. Task Scheduler 등록 사양

- 태스크명: `DailyScrum`
- 트리거: 매주 월~금 07:00
- 실행 계정: `SYSTEM` (ServiceAccount 로그온 타입, 최고 권한)
- 액션: 배치파일(`run_daily_scrum.bat`) 직접 실행 (cmd.exe 래퍼/외부 리다이렉션 없음)
- 실행 제한시간: 10분, 실패 시 1분 간격 최대 2회 재시도
- `-StartWhenAvailable`, `-RunOnlyIfNetworkAvailable` 옵션 적용

## 6. 향후 개선 아이디어
- 표 구조 이상 행에 대한 자동 보정 로직 (단, 잘못 보정 시 리스크가 있어 신중한 검토 필요)
- 실행 로그를 Confluence/외부 로그 시스템으로 중앙화하여 SYSTEM 계정 실행 환경에서도 상세 로그 확인 용이하게 개선
