# SIE 릴리즈 노트 자동화 도구 — 컨텐츠 기획서

## 1. 개요

SIE(Sony Interactive Entertainment)에서 수시로 발송되는 개발 공지(SDK/TRC/CertOps/PSN API/브랜드
가이드라인 변경 등)를 프로젝트 팀이 놓치지 않고 정리·공유하기 위한 데스크톱 도구다.

담당자가 공지 원문을 붙여넣으면:
1. AI가 변경 이력을 표 형태로 정리하고
2. Confluence의 "최근 변경 이력" 페이지를 갱신하며 (예전 이력은 자동으로 아카이브 페이지로 이동)
3. Teams 채널에 보기 좋은 공지문을 담당자 멘션과 함께 발송한다

## 2. 배경 / 문제 정의

- SIE 공지는 영어 원문 + 개발자 대상 전문 용어로 되어 있어, 그대로 공유하면 관련 없는 팀원까지
  다 읽어야 우리 프로젝트에 영향이 있는지 없는지 판단할 수 있었다.
- 공지가 뜰 때마다 사람이 직접 "우리 SDK 버전 기준으로 영향 있는지", "어느 팀이 확인해야 하는지"를
  판단해서 Confluence와 Teams에 수작업으로 옮기던 작업을, 반복 가능한 도구로 대체한다.
- 사람이 직접 옮기는 과정에서 생기는 실수(빠뜨림, 중복 공지, 서식 깨짐)를 줄인다.

## 3. 대상 사용자

- 프로젝트에서 SIE 공지를 모니터링하고 사내에 전파하는 담당 PM/엔지니어 (현재는 1인 운영 전제,
  로컬 실행 데스크톱 앱)

## 4. 핵심 사용 흐름

```
[공지 입력]                [분석]                [미리보기 & 발송]
공지 원문 붙여넣기   →   🔍 내용 분석 시작   →   변경 이력 테이블 확인/수정
(여러 건 동시 가능)        (Gemini API)            Teams 공지문 확인/수정
공지별 제목·날짜 입력                              ↓
파일 첨부(PDF/이미지)도                    📝 Confluence 업데이트  /  📢 Teams 발송
텍스트로 추출해 추가 가능                   (또는 🚀 전체 실행으로 한번에)
```

1. **공지 입력** — 공지 날짜, 공지 카드(제목/날짜/원문 텍스트)를 입력한다. 카드는 "＋ 공지 추가"로
   여러 개 만들 수 있고, 한 번에 여러 SIE 공지를 같이 분석할 수 있다. 원문이 PDF/이미지라면
   "📎 파일 첨부"로 텍스트를 추출해 새 카드로 넣을 수 있다.
2. **분석** — "🔍 내용 분석 시작"을 누르면 AI가 공지를 읽고 변경 이력 표, Confluence 반영이 필요한
   페이지/섹션, Teams 공지문 초안을 만들어준다. 현재 프로젝트 SDK 버전을 기준으로 "즉시 영향 /
   SDK 업그레이드 시 영향 / 해당 없음"을 판단해준다.
   - 이미 처리한 적 있는(또는 사실상 같은) 공지를 다시 분석하려 하면 미리 알려준다.
3. **미리보기 & 발송** — 두 개의 하위 화면이 있다.
   - **변경 이력 테이블**: AI가 정리한 결과를 직접 수정할 수 있다. 영역/버전/날짜/코드/상태/변경
     내용/영향 팀/확인 필요 사항을 항목별로 확인하고 고칠 수 있다.
   - **Teams 공지문**: 상단 인사말, 항목별 내용/조치내용/담당자, 마무리 문구를 확인·수정한다.
     담당자는 `@이름`을 치면 예전에 등록한 사람 목록이 자동완성으로 뜬다. 마무리 문구 안에서도
     `@이름`으로 멘션을 걸 수 있다.
4. **반영**
   - **📝 Confluence 업데이트**: "최근 변경 이력" 표를 새 내용으로 교체하고, 그 표에 있던 이전
     데이터는 자동으로 "버전별 아카이브" 페이지에 접이식(expand) 블록으로 옮겨 쌓아둔다. 이
     블록 제목은 이번에 반영한 공지들의 날짜를 기준으로 자동 생성된다 (예: `2026.07.20 까지의
     이력`, 여러 날짜가 섞여 있으면 `7월 첫째주 까지의 이력`처럼 뭉뚱그려 표시).
   - **📢 Teams 발송**: 실제 채널 또는 테스트 채널(체크박스로 전환)로 서식 있는 공지 카드를
     보낸다. 멘션 대상자는 Teams에서 실제로 호출 알림을 받는다.
   - **🚀 전체 실행**: 위 두 가지를 한 번에 실행한다.
5. **이력 관리** — 처리한 공지는 발송 이력에 자동 저장된다. "📋 발송 이력"에서 지난 처리 내역
   (처리 일시, 항목 미리보기, Confluence/Teams 반영 여부)을 확인할 수 있고, "📊 리포트"에서
   주간/월간 변경 이력을 Word 문서로 뽑아볼 수 있다.

## 5. 주요 기능 목록

| 영역 | 기능 |
|---|---|
| 공지 입력 | 여러 공지 동시 입력(카드형), 공지별 제목/날짜 개별 입력, PDF·이미지 첨부 시 텍스트 자동 추출 |
| AI 분석 | 변경 이력 자동 정리(영역/버전/상태/영향 팀/확인 사항), 현재 SDK 기준 영향도 판단, 기술 용어 괄호 설명 |
| 중복 방지 | 완전 동일 공지 재분석 경고, 사소한 복붙 잡음(공백·조회수 등)이 섞인 사실상 동일 공지도 유사도로 감지 |
| 변경 이력 테이블 | AI 분석 결과를 화면에서 직접 수정 후 그대로 Confluence 반영 |
| Confluence 자동화 | 최근 변경 이력 표 갱신 + 예전 이력 자동 아카이브(날짜 기반 제목 자동 생성), 실서비스/테스트 페이지 분리 |
| Teams 발송 | 서식 유지되는 카드형 메시지, 항목별 담당자 멘션(자동완성 지원), 마무리 문구 내 인라인 멘션, 테스트 채널 모드 |
| 발송 이력 | 처리 이력 조회, 재발송/재분석 시 참고 |
| 리포트 | 주간/월간 변경 이력을 Word(.docx)로 자동 생성 (영역별 통계, 상태별 통계, 상세 이력 포함) |

## 6. 현재 범위에 포함되지 않는 것

- 공지 원문 자동 수집(현재는 사람이 SIE 페이지에서 직접 복사해서 붙여넣는 방식)
- 발송/반영에 대한 승인 결재 절차 (담당자 1인이 확인 후 바로 반영)
- 예약 발송/자동 스케줄링 (사람이 직접 실행 버튼을 눌러야 동작)
- 다국어(영문 공지문 등) 지원

## 7. 최근 개선 사항 (참고)

- 발송 이력 팝업 스크롤 안 되던 문제 수정
- Teams 항목 편집 영역 스크롤 시 잔상(떠다니는 자동완성 창) 문제 수정
- 동일 공지 재분석 시 사소한 복붙 차이도 잡아내는 유사도 기반 중복 감지 추가
- 공지별 제목/날짜 개별 입력 기능 추가
- 변경 이력 테이블이 안내 문구와 달리 실제로는 수정이 안 되던 버그 수정
- Confluence 아카이브 제목을 오해 소지 없이 "OO 까지의 이력" 형태로 명확화

# SIE 릴리즈 노트 자동화 도구 — 상세 기술서

## 1. 아키텍처 개요

- **형태**: Python + Tkinter 기반 단일 데스크톱 GUI 앱 (Windows). `build_exe.bat` + PyInstaller로
  `dist/SIE_Processor.exe` 단일 실행 파일로 빌드해 배포.
- **실행 방식**: 로컬에서 사람이 직접 실행. 백그라운드 서비스/서버 없음. 네트워크 호출은 사용자가
  버튼을 눌렀을 때만 발생 (분석 실행, Confluence 업데이트, Teams 발송).
- **저장소**: 별도 DB 없이 로컬 JSON 파일 사용 (`send_history.json`, `mentions.json`) + `config.ini`.
- **동시성**: AI 분석(`analyze_notice`)은 UI 프리징 방지를 위해 `threading.Thread(daemon=True)`로
  백그라운드 실행 후 `root.after(0, ...)`로 메인 스레드에 결과 반영.

## 2. 모듈 구성

| 파일 | 책임 |
|---|---|
| `main.py` | Tkinter GUI 전체 (탭 구성, 이벤트 핸들러, 화면 상태 관리). 약 1900줄 규모. |
| `analyzer.py` | Gemini API(`google-genai`)로 공지 원문 분석 → 구조화된 JSON 반환 |
| `file_extractor.py` | PDF/이미지 → 텍스트 추출 (PyMuPDF 직접 추출 우선, 실패 시 Gemini Vision) |
| `confluence_client.py` | Confluence REST API 호출 (페이지 조회/갱신, 변경 이력 표·아카이브 HTML 조립) |
| `teams_client.py` | Teams Incoming Webhook 발송 (Adaptive Card 조립) |
| `mention_store.py` | `mentions.json` 기반 `@이름 → 이메일` 저장/조회/자동완성 |
| `history_store.py` | `send_history.json` 기반 발송 이력 저장, 중복/유사 공지 감지 |
| `report_generator.py` | `python-docx`로 주간/월간 리포트(.docx) 생성 |
| `config.ini` | 설정 (API 키, Confluence 페이지 ID, Teams 웹훅 URL 등 — 민감정보 포함, 외부 공유 금지) |

## 3. 외부 연동

### 3.1 Gemini API (`google-genai` SDK)
- 텍스트 분석: `analyzer.analyze_notice()` — 모델 폴백 순서
  `gemini-2.5-flash → gemini-2.5-flash-lite → gemini-3.5-flash → gemini-2.5-flash-preview-05-20`.
  429/404 계열 오류는 즉시 다음 모델로, 503(UNAVAILABLE)은 같은 모델로 최대 3회 재시도(5초/10초
  대기) 후 다음 모델로 넘어감.
- Vision(PDF/이미지 OCR성 추출): `file_extractor.py`에서 `gemini-2.5-flash`, `gemini-1.5-flash-latest`
  순으로 시도.
- 인증: `config.ini`의 `[gemini] api_key`.

### 3.2 Confluence REST API
- Basic Auth: `email:api_token`을 base64 인코딩해 `Authorization` 헤더로 전송.
- 사용 엔드포인트: `GET/PUT /wiki/rest/api/content/{page_id}` (storage representation).
- 대상 페이지 4종 (config.ini): 실서비스/테스트 각각의 `changelog_page_id`(최근 변경 이력),
  `archive_page_id`(버전별 아카이브). "테스트 모드" 체크 여부에 따라 어느 페이지 쌍을 갱신할지 결정.

### 3.3 Teams Incoming Webhook
- `config.ini`의 `[teams] webhook_url`(본채널) / `test_webhook_url`(테스트채널).
- 항상 Adaptive Card로 발송 (멘션 유무와 무관 — 과거엔 멘션 없으면 plain `{"text": ...}`로 보내
  서식이 깨졌던 문제가 있어 통일함).

## 4. 데이터 흐름

```
NoticeCard(제목, 날짜, 원문) × N
        │  _start_analysis()
        ▼
notices = [{title, date, content}, ...]  ── 중복/유사 감지(history_store.is_duplicate) ──▶ 경고 팝업
        │  analyzer.analyze_notice(api_key, notices, date, current_sdk)
        ▼
analyzed_data = {
  changelog_rows: [...],       # 변경 이력 표 항목
  confluence_pages: [...],     # (참고용, 자동 반영 대상 아님 — 사람이 확인)
  teams_notice: "..."          # Teams 공지문 초안
}
        │  _show_preview()
        ▼
UI 반영: changelog_text(직접 편집 가능) / teams_header,footer / teams_item_rows(내용·조치내용·담당자)
        │
        ├─ 📝 Confluence 업데이트
        │     rows = _get_rows_from_ui()  ← 화면에서 수정한 내용 재파싱
        │     confluence_client.update_changelog_and_archive(...)
        │
        └─ 📢 Teams 발송
              msg, mentions = _build_teams_message()  ← 항목별 담당자 + 마무리 문구 내 인라인 멘션 파싱
              teams_client.send_teams_notice(webhook_url, msg, mentions)

        │  (Confluence 반영 또는 Teams 발송 성공 시)
        ▼
history_store.save_history(make_entry(date, notices, analyzed_data, teams_sent, confluence_updated))
```

## 5. 데이터 모델

### 5.1 `notice` (분석 입력 단위)
```json
{"title": "공지 1 (또는 사용자 입력 제목)", "date": "2026.07.20", "content": "원문 텍스트"}
```

### 5.2 `changelog_row` (AI 분석 결과 / Confluence 표 1행)
```json
{
  "area": "SDK",
  "version": "SDK 14.00",
  "date": "2026.06.03",
  "code_item": "sceKernelFoo",
  "status": "🟢신규 | 🟡수정 | 🔴삭제",
  "change_content": "변경 내용 요약",
  "affected_teams": "프로그램 / PM",
  "action_required": "확인 필요 사항",
  "sdk_impact": "즉시영향 | SDK업그레이드시영향 | 해당없음"
}
```

### 5.3 변경 이력 테이블 화면 텍스트 포맷 (`main.py: _get_rows_from_ui` 파서와 1:1 대응)
화면(`changelog_text`)에 아래 형식으로 렌더링되며, 편집 후에도 이 라벨 문구를 유지해야
`Confluence 업데이트` 시 정상적으로 재파싱된다.
```
[1] 영역 │ 버전 │ 날짜 │ 코드/항목명
     상태: 🟢신규  영향팀: 프로그램 / PM
     내용: 변경 내용...
     확인: 확인 필요 사항...
     SDK: 즉시영향        (있을 때만)
```

### 5.4 `send_history.json` 항목
```json
{
  "datetime": "2026.07.20 13:22",
  "date": "2026.07.20",
  "notices_hash": "abc12345",
  "notices_normalized": "정규화된 원문 전체 (유사도 비교용)",
  "notices_preview": "code_item 상위 3개",
  "changelog_rows": [...],
  "teams_sent": true,
  "confluence_updated": true
}
```
최대 200건 유지(오래된 것부터 삭제).

### 5.5 `mentions.json`
```json
{"김하은": "eunokey0708@ncsoft.com", "...": "..."}
```

## 6. 핵심 로직 상세

### 6.1 중복/유사 공지 감지 (`history_store.py`)
1. 원문을 정규화(CRLF→LF, 줄 끝 공백 제거, 빈 줄 제거) 후 MD5 8자리 해시.
2. 해시가 완전히 같으면 즉시 중복.
3. 해시가 다르면 최근 30건에 한해 `difflib.SequenceMatcher` 유사도를 계산, `0.9` 이상이면
   "유사 공지"로 판단 (복붙 시 섞이는 조회수/타임스탬프 등 잡음 대응).

### 6.2 Confluence 아카이브 제목 생성 (`main.py: _notice_date_label`, `_update_confluence`)
- `_current_notices`의 개별 `date` 필드를 모아 대표 날짜 라벨을 만든다.
  - 날짜가 1개면 `YYYY.MM.DD` 그대로.
  - 여러 날짜가 같은 달이면 "몇째주"((day-1)//7 기준, 1~7일=첫째 ... 29~31일=다섯째)로 뭉뚱그림.
    같은 주면 `7월 첫째주`, 다른 주면 `7월 첫째~셋째주`.
  - 달까지 다르면 `2026.07.28 ~ 2026.08.02` 범위 표기.
- 최종 제목: `f"{date_label} 까지의 이력"`. (예전엔 버전명 + "(날짜 갱신)" 조합이었으나, "그 날짜에
  공지된 내용"으로 오해하기 쉬워 날짜 기준 "~까지의 이력" 문구로 단순화함.)

### 6.3 Confluence 표 교체/아카이브 (`confluence_client.update_changelog_and_archive`)
1. "최근 변경 이력" 페이지의 기존 데이터 행을 예시행(`(예시)` 텍스트 또는 노란색 스타일 포함)과
   실제 데이터행으로 분리.
2. 실제 데이터행이 있으면 "버전별 아카이브" 페이지의 `refined-expands` 매크로 안에 새
   `refined-expand`(제목 = 위 6.2 로직 결과) 블록으로 삽입, 번호 재정렬.
3. "최근 변경 이력" 페이지는 헤더행 + 예시행 유지 + 새 데이터행으로 tbody 전체 교체.
4. 두 페이지 모두 버전 번호(`version.number + 1`)를 올려 PUT.

### 6.4 Teams 멘션 처리 (`main.py`, `mention_store.py`, `teams_client.py`)
- 항목별 "담당자" 입력란: `@이름` 타이핑 시 `mentions.json`에서 접두어 검색해 자동완성 목록 표시,
  선택 시 `@이름/이메일` 형식으로 확정.
- 마무리 문구(footer, 여러 줄 Text)에도 동일한 자동완성 위젯 부착(`_setup_text_mention_autocomplete`).
- 발송 직전(`_build_teams_message`) footer 텍스트 내 `@이름/이메일` 토큰을 정규식으로 찾아
  `<at>이름</at>`으로 치환하고 mentions 리스트에 추가, 항목별 멘션과 합쳐 이메일 기준 중복 제거.
- `teams_client.send_teams_notice`는 멘션 유무와 관계없이 항상 Adaptive Card로 발송
  (`msteams.entities`에 `{type: mention, text: "<at>이름</at>", mentioned: {id: 이메일, name: 이름}}`).

### 6.5 스크롤 가능 영역 관리 (`main.py: _setup_global_scroll`)
- Tkinter `Canvas` + 내장 `Frame`으로 구현한 여러 스크롤 영역(공지 카드 목록, Teams 항목 목록,
  발송 이력 팝업)에 대해 개별 바인딩 대신, `_register_scrollable_canvas`로 등록한 Canvas 목록을
  `root.bind_all("<MouseWheel>")` 전역 핸들러 하나가 위젯 계층을 타고 올라가며 찾아 스크롤한다.
  새 스크롤 영역을 추가할 땐 반드시 이 함수로 등록해야 마우스 휠이 동작한다.
- 팝업(Toplevel)처럼 수명이 짧은 스크롤 영역은 `<Destroy>` 이벤트에 맞춰
  `_unregister_scrollable_canvas`로 정리한다.
- 항목별 담당자 자동완성 `Listbox`는 z-order 문제로 `root`에 직접 배치하는데, 소유 행(`PanedWindow`)의
  `<Destroy>` 이벤트에 맞춰 함께 파괴하도록 바인딩해 재분석 시 위젯이 누적되지 않게 한다.

## 7. 설정 파일 구조 (`config.ini`)

```ini
[gemini]
api_key = <GEMINI_API_KEY>

[confluence]
base_url = https://nckorea.atlassian.net
email = <계정 이메일>
api_token = <CONFLUENCE_API_TOKEN>
changelog_page_id = <테스트/기본 최근 변경 이력 페이지 ID>
archive_page_id = <테스트/기본 버전별 아카이브 페이지 ID>
dev_changelog_page_id = <실서비스 최근 변경 이력 페이지 ID>
dev_archive_page_id = <실서비스 버전별 아카이브 페이지 ID>

[teams]
webhook_url = <본채널 Incoming Webhook URL>
test_webhook_url = <테스트채널 Incoming Webhook URL>

[ui]
theme = dark | light

[project]
current_sdk = <현재 빌드 머신 SDK 버전, 예: 12.00.00>
```
> `api_key`/`api_token`/`webhook_url`은 실제 인증 정보이므로 이 문서 및 저장소 외부 공유 금지.

## 8. 알려진 제약 / 향후 개선 여지

- `confluence_pages`(AI가 제안하는 "추가로 수정해야 할 Confluence 페이지/섹션") 항목은 현재
  화면에 노출되거나 자동 반영되지 않는 필드로, 실질적으로 사용되지 않는 응답 필드다.
- 발송 이력은 최대 200건만 보관되며 그 이상은 오래된 순으로 삭제된다(별도 백업 없음).
- Gemini 응답이 JSON 형식을 벗어나면(`json.loads` 실패) 예외로 처리되어 분석이 실패한다 —
  재시도는 모델 단위(429/503)에 한하고, JSON 파싱 실패 자체에 대한 재시도 로직은 없다.
- 완전한 오프라인 큐잉/재시도는 없음 — 네트워크 실패 시 사용자가 버튼을 다시 눌러야 한다.
