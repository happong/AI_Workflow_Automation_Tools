#  빌드 머신 자산 관리 시스템 — 컨텐츠 기획서

## 1. 개요

빌드 머신(PC), 모바일 기기, PS(플레이스테이션) 자산을 한 곳에서 등록·조회·관리하고, 실시간 가동 상태(CPU/GPU/디스크 사용률, 온라인 여부)까지 자동으로 파악할 수 있는 사내 웹 서비스.

- 웹 서버 주소: http://localhost:3000
- 개발: PM팀 김하은, Claude(AI)와의 협업으로 개발/반복 개선

## 2. 문제 정의

기존에는 Confluence 위키 페이지에 자산 정보를 수기로 정리했으나 다음 문제가 있었음:

- 수작업 업데이트로 정보 지연·오류 발생
- 자산 소유자/담당자 불명확으로 관리 공백
- IP/MAC 등 네트워크 정보가 분산 관리되어 조회 불편
- 자산 추가/변경 요청 및 승인 프로세스 부재
- 빌드 머신 스펙 정보 수집이 수작업
- 머신이 실제로 사용 중인지, 부하가 어느 정도인지 알 방법이 없음 (물어보는 수밖에 없었음)

## 3. 대상 사용자

| 유형 | 설명 |
|---|---|
| 관리자 | 자산 승인/거절, 정보 수정·삭제, 접근권한 관리, Confluence/Jenkins 연동 설정 |
| 일반 사용자 (AIR FRAME 계정) | 자산 조회, 자산 추가 요청, 본인 요청 현황 확인 |
| 빌드 머신 소유자 | PC 사양 수집 스크립트 실행, 모니터링 에이전트 설치 |
| 모바일 기기 소유자 | 모바일 정보 수집 페이지 접속 후 자산 등록 |

## 4. 핵심 기능

### 4-1. 자산 관리
- PC / 모바일 / PS 세 종류로 자산을 구분하여 탭으로 분리 관리
- 카드형 UI: IP 온라인 여부, 자산번호(호스트명), 실시간 CPU/GPU/드라이브별 사용률 막대그래프, 용도, 상태(사용중/미정/확인 필요)를 한눈에 표시
- 자산번호/호스트명/소유자/용도 기준 정렬, 상태별 필터링
- 카드 클릭 시 상세 정보(스펙, 네트워크, 실시간 모니터링, 과부하 여부, 최종 점검일) 확인 및 스펙 텍스트 직접 편집 가능

### 4-2. 자산 추가/승인 워크플로우
- 누구나 "자산 추가 요청" 가능 (기기 종류 선택: PC/모바일/PS)
- PC는 사양 수집 스크립트로 만든 txt 파일을 업로드하면 호스트명/사양/IP/MAC/Storage/네트워크 속도가 자동 입력됨
- 관리자 승인 시 자산 목록에 반영, 첨부했던 스펙 파일도 함께 저장되어 상세 페이지에서 그대로 확인 가능
- 거절 시 사유 기록, 요청 히스토리 영구 보존

### 4-3. 실시간 모니터링
- PC에 PowerShell 에이전트를 설치하면 5분마다 CPU/GPU/드라이브별 디스크 사용률을 자동 보고
- MAC 주소 기준으로 자산과 매칭 (호스트명 변경에 영향받지 않음)
- 서버가 자체적으로 네트워크 서브넷을 주기적으로 스캔해 온라인 여부·MAC·최종 점검일 자동 갱신, 미등록 기기 탐지
- Jenkins 빌드 노드 상태(온라인/유휴/가동 여부) 조회

### 4-4. 모바일 자산 정보 수집
- 설치 없이 모바일 브라우저로 페이지에 접속하면 OS/해상도(및 안드로이드는 네트워크 타입)를 자동 수집
- 제품명/모델명/시리얼번호/MAC 등 브라우저 보안 정책상 자동 수집이 불가능한 항목은 직접 입력
- 결과를 txt로 저장해 자산 추가 요청 시 첨부

### 4-5. Confluence 연동
- 자산 표를 클립보드로 복사해 Confluence에 붙여넣기 (기존 기능)
- **자동 동기화 버튼** (신규): 관리자 페이지에서 버튼 클릭 한 번으로
  - 서버실 소재 PC 자산 표, 사용중 상태 PC 자산 표를 최신 데이터로 자동 갱신
  - 갱신 전 표는 Expandable panel(아코디언)에 날짜별로 자동 보관되어 이력 추적 가능
  - 테스트 페이지에서 먼저 검증 후 운영 페이지에 반영하는 2단계 안전장치

### 4-6. 관리자 도구
- 접근권한 관리 (관리자/화이트리스트/그룹 기반 접근 제어, AIR FRAME 연동)
- 빌드 머신 설정 가이드 및 스크립트(PC 스펙 수집기, 모니터링 에이전트) 다운로드 배포
- Confluence 동기화 실행

## 5. 사용자 시나리오

**시나리오 A — 신규 빌드 머신 등록**
1. 담당자가 포털에서 "빌드 머신 설정 가이드" 다운로드
2. 가이드에 따라 스펙 수집 스크립트 실행 → txt 생성
3. "자산 추가 요청"에서 txt 업로드 → 자동 입력된 정보 확인 후 제출
4. 관리자 승인 → 자산 목록에 카드로 표시
5. 모니터링 에이전트 설치 → 5분마다 자동으로 CPU/GPU/디스크 정보 갱신

**시나리오 B — 정기 Confluence 보고**
1. 관리자가 관리자 페이지 → 정보수집 탭 접속
2. "테스트 페이지 업데이트"로 먼저 확인
3. 문제없으면 "운영 페이지 업데이트" 클릭
4. Confluence 페이지의 자산 표가 최신 상태로 갱신되고, 이전 표는 날짜별로 보관됨

**시나리오 C — 모바일 자산 등록**
1. 모바일 브라우저로 "모바일 정보 수집" 페이지 접속
2. 자동 수집된 정보 확인, 나머지 직접 입력
3. 자산 추가 요청 시 기기 종류를 "모바일"로 선택하고 등록

## 6. 알려진 한계

- 사내망이 HTTP(비HTTPS)로 운영되어, 모바일에서 기기 모델명 등 일부 브라우저 API(User-Agent Client Hints)는 사용 불가 — 보안 정책상 HTTPS 필수 기능이기 때문
- iOS Safari는 안드로이드 대비 자동 수집 가능한 정보가 더 적음 (기기 모델명, 저장공간, 네트워크 타입 등 대부분 미지원)
- 실시간 모니터링은 에이전트를 설치한 머신에서만 동작 (설치 전까지는 "미확인" 상태 유지)
- `data.json`(자산 데이터), `users.json`(자격 증명)은 P4 형상관리 대상에서 제외 — 운영 서버 값을 덮어쓰지 않기 위한 의도적 결정이며, 신규 설정 항목은 운영 서버에 수동으로 반영 필요

## 7. 향후 고려사항 (제안)

- HTTPS 적용 시 모바일 자동 수집 범위 확대 가능
- ADB 기반 안드로이드 정밀 수집(모델명/실제 저장공간) 옵션 추가 검토
- Confluence 자동 동기화 스케줄링(수동 버튼 외 정기 자동 실행) 검토

#  빌드 머신 자산 관리 시스템 — 상세 기술 기술서

## 1. 아키텍처 개요

- **런타임**: Node.js + Express (단일 프로세스, 단일 서버)
- **데이터 저장**: 별도 DB 없이 JSON 파일 기반 (`data.json`, `users.json`)
- **프론트엔드**: 별도 빌드 과정 없는 순수 HTML/CSS/JS (index.html, admin.html, login.html, mobile-spec.html 전부 단일 파일에 인라인)
- **인증**: express-session 기반 서버 세션 + AIR FRAME(사내 SSO 유사 시스템) API 키 검증
- **배포**: Windows 서비스로 상시 구동 (node-windows, `install-service.js`), 또는 `run_server.vbs`로 백그라운드 실행
- **형상관리**: Perforce(P4V). `data.json`/`users.json`은 환경별 실데이터/자격증명이라 P4 추적 대상에서 제외

## 2. 디렉토리 구조

```
_Build_manager/
├── server.js                 # 메인 서버 (Express, 전체 API)
├── netscan.js                 # 서브넷 ping 스캔 + MAC/최종점검일 갱신
├── jenkins.js                  # Jenkins REST API 연동
├── confluence.js               # Confluence REST API 연동 (표 자동 동기화)
├── data.json                   # 자산/대기요청/히스토리 데이터 (P4 제외)
├── users.json                  # 관리자/화이트리스트/외부 연동 자격증명 (P4 제외)
├── collect_spec.ps1            # PC 사양 수집 스크립트 (UTF-8 BOM 필수)
├── run_collect_spec.bat        # 위 스크립트를 서명오류 없이 실행하는 더블클릭용 배치
├── 빌드머신_정보수집_가이드.md   # 신규 머신 등록 가이드 (포털에서 다운로드 가능)
├── agent/
│   ├── report-agent.ps1        # CPU/GPU/디스크 사용률을 서버로 보고 (1회 실행)
│   ├── install-task.ps1        # 위 스크립트를 5분마다 실행하는 예약 작업 등록
│   └── test-agent-local.bat    # 로컬 개발 서버(localhost)로 테스트 전송하는 더블클릭용 배치
├── specs/
│   └── asset_<id>.txt          # 자산별 원본 스펙 텍스트 파일
└── public/
    ├── index.html               # 메인 포털 (자산 목록/카드 UI)
    ├── admin.html                # 관리자 페이지 (접근권한, 정보수집 도구, Confluence 동기화)
    ├── login.html                # 로그인 페이지
    └── mobile-spec.html          # 모바일 기기 정보 수집 페이지 (로그인 불필요)
```

## 3. 데이터 모델 (`data.json`)

```jsonc
{
  "assets": [
    {
      "id": 1,                      // 서버 내부 고유 id (nextId로 순증)
      "assetNo": "HPC0A-XXXXXXX",   // 자산 관리번호
      "deviceType": "pc",            // "pc" | "mobile" | "ps"
      "host": "호스트명",
      "owner": "소유자", "ownerVerified": true,
      "user": "사용자",
      "spec": "CPU/RAM/GPU 요약 문자열 (모바일은 제품명, PS는 미사용)",
      "storage": "디스크 용량 요약 (모바일은 모델명)",
      "purpose": "용도",
      "status": "사용중 | 미정 | 확인 필요",
      "location": "위치 (예:  서버실, 소유주자리 등 — 공백 유무 비일관 데이터 존재)",
      "adminId": "관리자 ID",
      "ip": "", "mac": "", "network": "", "os": "",
      "section": "구분 라벨",
      "lastCheckedAt": "ISO8601",           // netscan.js가 주기 갱신
      "liveStats": {                        // report-agent.ps1이 갱신
        "cpuPercent": 0, "gpuPercent": 0,
        "disks": [{ "drive": "C:", "usedPercent": 45.4 }],
        "updatedAt": "ISO8601"
      }
    }
  ],
  "pending": [ /* 자산 추가 요청 대기열, approve/reject 전까지 보관 */ ],
  "history": [ /* 승인/거절 완료된 요청 이력 (result, reason, resolvedAt 포함) */ ],
  "nextId": 52,
  "unregisteredHosts": [ /* netscan.js가 찾은, 자산에 등록 안 된 응답 IP */ ]
}
```

`location` 필드는 과거 데이터 이관 과정에서 `" 서버실"`(공백 있음)과 `"서버실"`(공백 없음)이 혼재하므로, 이 값으로 필터링할 때는 반드시 `.replace(/\s+/g,'')` 정규화 후 비교해야 함.

## 4. `users.json` (자격증명/설정 — P4 제외)

| 필드 | 용도 |
|---|---|
| `admins`, `whitelist`, `allowedGroups` | 접근권한 (관리자 즉시 접근, 사용자는 화이트리스트/그룹 필요) |
| `managerApiKey` | AIR FRAME 그룹 조회용 폴백 API 키 |
| `agentSecret` | `/api/agent/report` 인증용 공유 비밀값 (헤더 `X-Agent-Secret`) |
| `jenkinsUrl`, `jenkinsUsername`, `jenkinsApiToken` | Jenkins Basic Auth |
| `confluenceBaseUrl`, `confluenceEmail`, `confluenceApiToken` | Confluence Basic Auth (API 토큰 방식) |
| `confluenceTestPageId`, `confluenceProdPageId` | 동기화 대상 페이지 ID |

신규 필드를 추가할 때마다 **운영 서버의 `users.json`에도 수동으로 반영**해야 함 (P4로 자동 전파되지 않음).

## 5. API 엔드포인트

### 인증/접근권한
| Method | Path | 권한 | 설명 |
|---|---|---|---|
| POST | `/api/auth/login` | - | AIR FRAME API 키로 로그인, 관리자/화이트리스트/그룹 순으로 접근 판정 |
| POST | `/api/auth/logout` | - | 세션 종료 |
| GET | `/api/auth/me` | - | 현재 로그인 사용자 조회 |
| GET/POST/DELETE | `/api/whitelist`, `/api/admins`, `/api/groups` | admin | 접근권한 CRUD |
| POST | `/api/access/set-role` | admin | 관리자↔사용자 역할 변경 |
| GET | `/api/air/user/:userId`, `/api/air/groups`, `/api/air/group-members/:groupId` | admin | AIR FRAME 조회 프록시 |
| POST | `/api/settings/manager-key` | admin | `managerApiKey` 설정 |

### 자산
| Method | Path | 권한 | 설명 |
|---|---|---|---|
| GET | `/api/assets` | auth | 전체 자산 + 대기요청 목록 |
| GET/POST/DELETE | `/api/assets/:id/spec` | auth/admin | 스펙 txt 조회/업로드/삭제 |
| PUT | `/api/assets/:id` | admin | 자산 필드 수정 |
| DELETE | `/api/assets/:id` | admin | 자산 삭제 (스펙 파일도 함께 삭제됨) |
| POST | `/api/pending` | auth | 자산 추가 요청 생성 (PC는 `specFileContent` 첨부 시 승인 시점에 스펙 파일로 저장됨) |
| GET | `/api/pending/by/:requester`, `/api/history/by/:requester` | auth | 요청자별 조회 |
| POST | `/api/pending/:id/approve` | admin | 승인 → 자산 생성 (모바일/PS는 스펙 txt 자동 생성, PC는 첨부 파일 그대로 저장) |
| DELETE | `/api/pending/:id` | admin | 거절 |
| DELETE | `/api/history/:id`, `/api/history` | admin | 히스토리 삭제 |

### 모니터링/연동
| Method | Path | 권한 | 설명 |
|---|---|---|---|
| GET | `/api/status/ping` | auth | 온라인/오프라인 캐시 조회 (1분 주기 자동 갱신) |
| GET | `/api/network/unregistered` | admin | 서브넷 스캔으로 발견된 미등록 기기 |
| GET | `/api/status/jenkins` | auth | Jenkins 노드 상태 (1분 주기 폴링) |
| POST | `/api/agent/report` | agentSecret | 에이전트 CPU/GPU/디스크 리포트 수신, MAC 기준 매칭 |
| GET | `/api/status/hostname/:ip` | auth | 역방향 DNS 조회 |
| POST | `/api/confluence/sync-assets` | admin | Confluence 표 자동 동기화 (`target: 'test'\|'prod'`) |

### 정적/다운로드
- `/`, `/login`, `/admin`, `/mobile-spec` — 페이지 라우팅
- `/downloads/collect_spec.ps1`, `/downloads/run_collect_spec.bat`, `/downloads/guide.md`, `/downloads/agent/*` — 빌드 머신 배포용 파일 (P4V 없이 웹 다운로드)

## 6. 모니터링 에이전트 동작 원리

1. `install-task.ps1` 1회 실행 → Windows 작업 스케줄러에 `-AssetAgent-Report` 등록 (5분 간격, `RepetitionDuration`은 `[TimeSpan]::MaxValue` 대신 10년(3650일)로 설정 — MaxValue는 Task Scheduler XML 스키마 범위를 초과해 등록 실패하는 알려진 이슈가 있음)
2. `report-agent.ps1`이 5분마다 실행되어:
   - `Get-CimInstance Win32_Processor`로 CPU 사용률
   - `nvidia-smi`로 GPU 사용률 (NVIDIA 전용, 실패해도 무시)
   - `Get-Volume`으로 드라이브 문자 있는 고정 볼륨만 사용률 수집 (드라이브 문자 없는 시스템/복구 파티션은 의도적으로 제외)
   - 활성 네트워크 어댑터의 MAC 주소
   - 위 정보를 `X-Agent-Secret` 헤더와 함께 서버 기본값(`http://172.18.20.33:3000`)으로 POST
3. 서버는 **MAC 주소 정규화 비교**(대소문자/구분자 무시)로 자산을 매칭 — 호스트명 변경이나 IP 변경에 영향받지 않음
4. 로컬 개발 서버로 테스트하려면 `test-agent-local.bat` 사용 (`-ServerUrl http://localhost:3000` 고정)

## 7. 외부 연동

### Jenkins (`jenkins.js`)
- Basic Auth (`username:apiToken`)
- `GET /computer/api/json?tree=computer[displayName,offline,idle,executors[...]]` 로 노드 상태 조회, 1분 캐시
- 주의: Jenkins 노드의 `displayName`이 자산의 `host`와 이름 규칙이 다를 수 있어(역할 기반 vs 머신명 기반) 자동 매칭은 하지 않음 — 필요 시 수동 매핑 추가 필요

### Confluence (`confluence.js`)
- Basic Auth (`email:apiToken`, Atlassian API 토큰)
- Confluence Cloud REST API v1 사용 (`/wiki/rest/api/content/{id}`) — 2025년 이후 지원 종료 예정(Deprecation 헤더 확인됨), 추후 v2 API로 마이그레이션 필요
- 페이지의 `body.storage`(Confluence Storage Format, XHTML 유사)를 문자열 검색/슬라이스 방식으로 직접 조작 — 별도 XML 파서 미사용
- 동작:
  1. `자산 기본 정보`, `사용 중 자산` 두 제목 뒤의 첫 `<table>`을 각각 탐색
  2. 두 표를 `<h3>` 라벨과 함께 하나의 `refined-expand`(Refined 앱의 Expand 매크로, TOC 미노출을 위해 h3 사용) 항목으로 묶어 `refined-expands` 그룹의 맨 앞에 삽입 (날짜 제목)
  3. 원래 위치에는 최신 데이터로 생성한 새 표로 교체 (자산 기본 정보: 서버실 소재 PC 10컬럼 / 사용 중 자산: 사용중 상태 PC 9컬럼)
  4. `version.number + 1`로 PUT하여 갱신
- 페이지 구조가 바뀌면(제목 텍스트, 표 순서, refined-expands 매크로 존재 여부) 파싱이 실패하므로, 구조 변경 시 `confluence.js`의 `findTableAfterHeading`/`transformBody` 수정 필요

## 8. 프론트엔드 주요 로직 (`public/index.html`)

- **카드 렌더링**: `renderAssetTable()` → 기기 종류 탭(`devtype-btn`)으로 PC/모바일/PS 필터링 후 카드 그리드 생성
- **정렬**: `sort-field` select 값 기준 `localeCompare(..., 'ko')` 오름차순
- **실시간 통계 바**: `statBarRow()`가 CPU/GPU/드라이브별 사용률을 단일 색상(평시 accent, 70%↑ yellow, 90%↑ red) 바로 표시 — 과거 버전은 사용량/잔여량을 초록/빨강 두 색으로 나눠 표시했으나 시각 피로 이슈로 변경됨
- **스펙 상세/편집**: `openSpecModal()`이 자산 요약 카드 + 실시간 모니터링 값을 렌더링하고, `<details>` 아코디언 안에서 원본 스펙 txt를 직접 편집(`saveSpecTxt()`) 가능. 저장 시 기기 종류별로 텍스트를 파싱해 실제 자산 필드(`spec`/`storage`/`os`/`network`/`mac`/`ip`)에도 반영한 뒤 `PUT /api/assets/:id`로 영구 저장 — 이 반영 로직이 빠지면 화면에는 보여도 새로고침 시 사라지는 버그가 재현되므로 유지 필요
- **자산 추가 요청 폼**: PC 스펙 txt 업로드 시 `parseSpecFile()`이 필드 자동 채움 + 원본 텍스트를 `specFileContent`로 보관해 요청 제출 시 함께 전송 (승인 시 서버가 `specs/asset_<id>.txt`로 저장)
- **네트워크 속도 파싱 주의**: 원본 txt의 `"속도 : 1 Gbps"`처럼 숫자와 단위 사이 공백이 있으므로, 드롭다운 값(`"1G"`)과 매칭하려면 숫자만 추출해 재조합해야 함 (문자열 치환만으로는 공백이 남아 매칭 실패)

## 9. 배포/운영 노트

- 로컬 개발: `node server.js`로 직접 실행, `http://localhost:3000`
- 운영: `http://172.18.20.33:3000`, Windows 서비스(`install-service.js`)로 상시 구동
- 배포 절차: 로컬 수정 → 테스트 → P4V Submit → 운영 서버 파일 반영 → 서비스 재시작 → **`users.json` 신규 필드 수동 반영**
- `data.json`/`users.json`은 P4 미추적이므로 로컬 파일이 읽기 전용으로 동기화되는 경우가 있어(P4 clobber 동작), 수정 전 `chmod +w` 또는 P4V에서 쓰기 가능 상태 확인 필요

# 빌드 머신 정보 수집 가이드

이 문서는 빌드 머신(개인 PC/서버)에서 사양 정보를 수집하고, 자산관리 포털에 등록한 뒤, CPU/GPU/디스크 사용률이 자동으로 갱신되도록 설정하는 방법을 안내합니다.

> P4V로 파일을 받지 않는 머신을 위한 가이드입니다. 필요한 스크립트는 모두 자산관리 포털 서버에서 웹으로 바로 다운로드합니다.

- 자산관리 포털: http://172.18.20.33:3000
- 이 가이드 다운로드: http://172.18.20.33:3000/downloads/guide.md

---

## 0. 전체 흐름

1. PC 사양 정보 수집 (`collect_spec.ps1`)
2. 포털에 자산 등록 (이미 등록되어 있다면 생략)
3. 실시간 모니터링 에이전트 설치 (`report-agent.ps1` + `install-task.ps1`)
4. 포털에서 확인

---

## 1. PC 사양 정보 수집

### 1-1. 다운로드

빌드 머신에서 브라우저로 아래 두 파일을 다운로드합니다. **어디에 저장되는지 신경 쓰지 마세요** — 보통 `다운로드`(Downloads) 폴더에 그대로 저장되면 됩니다. 별도 폴더로 옮길 필요 없습니다.

- http://172.18.20.33:3000/downloads/collect_spec.ps1
- http://172.18.20.33:3000/downloads/run_collect_spec.bat

### 1-2. 실행

다운로드 폴더를 열어서 `collect_spec.ps1`이 아니라 **`run_collect_spec.bat`을 더블클릭**하세요.

- ⚠️ **터미널(PowerShell/cmd)에 `powershell -File .\collect_spec.ps1` 같은 명령어를 직접 치지 마세요.** 지금 터미널이 열려있는 위치와 파일이 실제로 저장된 위치(보통 `다운로드` 폴더)가 다르면 "파일을 찾을 수 없다"는 오류가 납니다. `.bat`을 더블클릭하면 이런 경로 문제 자체가 없습니다 — 파일이 어느 폴더에 있든 알아서 자기 위치를 찾아 실행되도록 만들어져 있어요.
- `.ps1`을 직접 실행하면 "디지털 서명이 없어 실행할 수 없습니다" 오류가 날 수 있습니다. `.bat`은 내부적으로 `-ExecutionPolicy Bypass` 옵션으로 실행하므로 이 문제도 없습니다.

완료되면 바탕화면에 `spec_호스트명.txt` 파일이 생성됩니다. 이 안에 CPU/RAM/GPU/디스크/네트워크 정보가 정리되어 있습니다.

---

## 2. 자산 등록 (아직 포털에 등록 안 된 머신만)

1. 포털 접속 → 우측 상단 **＋ 자산 추가 요청** 클릭
2. "📄 스펙 txt 파일로 자동 입력"에 위에서 만든 `spec_호스트명.txt`를 업로드하면 CPU/RAM/GPU/디스크 항목이 자동으로 채워집니다.
3. 나머지 필수 항목(요청자 이름, 자산번호, 자산 소유자, 관리자 ID, **IP**, **MAC**)을 입력합니다.
   - **MAC 주소는 정확히 입력해야 합니다.** 3단계 모니터링 에이전트가 MAC으로 자산을 찾기 때문에, MAC이 다르면 리포트가 연결되지 않습니다.
   - MAC은 `spec_호스트명.txt`의 [네트워크 어댑터] 항목에서 확인할 수 있습니다.
4. 요청 제출 → 관리자 승인 후 자산 목록에 표시됩니다.

이미 등록된 머신이라면 이 단계는 건너뛰고 3단계로 이동하세요.

---

## 3. 실시간 모니터링 에이전트 설치

CPU/GPU/드라이브별 디스크 사용률을 5분마다 자동으로 포털에 보고하는 에이전트입니다.

### 3-1. 다운로드

```powershell
mkdir C:\-agent
Invoke-WebRequest http://172.18.20.33:3000/downloads/agent/report-agent.ps1 -OutFile C:\-agent\report-agent.ps1
Invoke-WebRequest http://172.18.20.33:3000/downloads/agent/install-task.ps1 -OutFile C:\-agent\install-task.ps1
```

(브라우저로 직접 다운로드해도 됩니다: http://172.18.20.33:3000/downloads/agent/report-agent.ps1 , http://172.18.20.33:3000/downloads/agent/install-task.ps1 — 두 파일을 반드시 같은 폴더에 둡니다.)

### 3-2. 설치 (1회만 실행) — 두 가지 방식 중 선택

| 방식 | 실행 권한 | 특징 |
|---|---|---|
| 일반 계정 (기본) | 관리자 권한 불필요 | 이 계정이 로그아웃하면 자동 리포트가 중단될 수 있음 |
| SYSTEM 계정 | **관리자 권한으로 PowerShell 실행 필요** (설치 시 1회만) | 로그인 상태·계정과 무관하게 항상 자동 실행됨 |

로컬 관리자 권한이 있다면 **SYSTEM 방식을 추천**합니다. 권한이 없다면 일반 계정 방식으로 진행하세요.

**일반 계정으로 설치:**
```powershell
powershell -ExecutionPolicy Bypass -File C:\-agent\install-task.ps1
```

**SYSTEM 계정으로 설치** (PowerShell을 "관리자 권한으로 실행"한 뒤):
```powershell
powershell -ExecutionPolicy Bypass -File C:\-agent\install-task.ps1 -System
```

"등록 완료: -AssetAgent-Report (5 분마다 실행)" 메시지가 뜨면 완료입니다. 이후로는 사람이 아무것도 안 해도 5분마다 자동으로 리포트가 올라갑니다.

### 3-3. 지금 바로 한 번 테스트하고 싶다면

```powershell
powershell -ExecutionPolicy Bypass -File C:\-agent\report-agent.ps1
```

"리포트 전송 완료: ..." 메시지가 뜨면 정상입니다.

---

## 4. 확인

포털 자산 목록에서 해당 머신 카드를 확인합니다.

- 카드 오른쪽에 CPU / GPU / 드라이브별(C:, D: ...) 사용률 막대가 표시됩니다.
- 카드를 클릭하면 상세 정보(과부하 여부, 최종 점검일 등)를 볼 수 있습니다.

---

## 5. 문제 해결

| 증상 | 원인 / 해결 |
|---|---|
| `.ps1` 실행 시 "디지털 서명이 없어 실행할 수 없습니다" | `.bat` 파일로 실행하거나, `powershell -ExecutionPolicy Bypass -File 파일명.ps1`로 직접 실행 |
| 리포트 전송 시 "자산 목록에서 MAC 'XX-XX-...'를 찾을 수 없습니다" | 포털에 등록된 자산의 MAC 값과 이 PC의 실제 MAC이 다름. 2단계에서 등록한 MAC을 다시 확인 |
| 카드에 CPU/GPU만 나오고 디스크가 안 나옴 | 에이전트를 처음 설치한 직후라 아직 실행 전인 경우가 많음. `report-agent.ps1`을 한 번 수동 실행해서 즉시 반영 가능 |
| 스크립트 다운로드 URL이 404 | 포털 서버가 아직 재배포 전일 수 있음. 담당자(eunokey0708)에게 문의 |
| 콘솔에 한글이 깨져서 나옴 | 스크립트 파일이 BOM 없는 UTF-8로 저장된 경우 발생. 담당자에게 재배포 요청 |
| `-File 매개 변수에 대한 인수 '.\collect_spec.ps1'이(가) 없습니다` | 터미널이 열려있는 폴더와 파일이 실제로 있는 폴더(보통 `다운로드`)가 다름. `.bat` 더블클릭으로 실행하거나, `cd Downloads` 후 재실행, 또는 전체 경로(`powershell -File "C:\Users\계정명\Downloads\collect_spec.ps1"`)로 실행 |
| `install-task.ps1 -System` 실행 시 "관리자 권한으로 실행한 뒤 다시 시도해주세요" | PowerShell을 일반 권한으로 실행함. 시작 메뉴에서 PowerShell 우클릭 → "관리자 권한으로 실행" 후 재시도 |
