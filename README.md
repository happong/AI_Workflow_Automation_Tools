# AI_Workflow_Automation_Tools
AI를 활용한 툴 생성한 내용입니다. 
# 🤖 AI Workflow Automation Tools

> 게임 개발 프로젝트(JSY)에서 반복되는 수작업을 AI API로 자동화한 도구 모음입니다.  
> A collection of AI-powered tools built to eliminate repetitive manual work in game development workflows.

---

## 📌 배경 (Background)

게임 개발 프로젝트를 운영하면서 매일 반복되는 수작업들이 있었습니다.

- 수십 MB 빌드 실패 로그를 사람이 직접 읽고 원인 파악
- 영문 SIE(Sony Interactive Entertainment) 공지를 수동으로 번역·요약·전파
- 매일 아침 팀 데일리 스크럼 페이지를 수기로 생성
- 빌드 머신 자산 현황을 파악할 통합 관리 도구 부재

이 도구들은 위 문제들을 **Claude / Gemini / GPT API**와 **Confluence / Teams / Jenkins API**를 연결해 해결한 결과물입니다.

---

## 🛠️ 도구 목록 (Tools)

### 1. 빌드 로그 AI 분석 대시보드
**Build Log AI Analysis Dashboard**

| 항목 | 내용 |
|------|------|
| 문제 | 빌드 실패 시 수십 MB 로그를 수동 분석 → 원인 파악까지 장시간 소요 |
| 해결 | Claude/GPT/Copilot API 연동으로 에러 자동 분류·담당자 라우팅·패턴 학습 |
| 기술 | Claude API, GPT API, Jenkins API, Node.js |
| 성과 | 반복 에러 패턴 통계화, 이력 검색, 심각도(Critical/Warning/Info) 자동 분류 |

**주요 기능**
- 빌드 파이프라인별(PS5/Package/CookValidation 등) 실패 원인 카테고리 통계
- 동일 에러 패턴 TOP 5 식별 및 재발 방지 근거 제공
- 에러 해결 방법 로컬 학습 → 동일 패턴 재발 시 1순위 가이드 제공
- 이전 빌드 대비 새로 생긴 에러 하이라이트
- Jenkins REST API 연동으로 로그 자동 수집 (온디맨드)

---

### 2. SIE 공지 자동화 툴
**SIE Notice Automation Tool**

| 항목 | 내용 |
|------|------|
| 문제 | 영문 SIE 공지(SDK/TRC/PSN API 변경 등)를 수동 번역·요약·Confluence+Teams 전파 |
| 해결 | Gemini API로 공지 자동 분석, 영향도 판단, Confluence 갱신 + Teams 발송 자동화 |
| 기술 | Gemini API, Confluence API, Teams Webhook, Python |
| 성과 | 공지 전파 수작업 100% 제거, 중복 공지 유사도 감지, 주간/월간 리포트 자동 생성 |

**주요 기능**
- 현재 프로젝트 SDK 버전 기준 "즉시 영향 / 업그레이드 시 영향 / 해당 없음" 자동 판단
- Confluence "최근 변경 이력" 자동 갱신 + 이전 이력 자동 아카이브
- Teams 채널 서식 있는 카드형 공지 + 담당자 멘션 자동 발송
- PDF/이미지 첨부 시 텍스트 자동 추출
- 유사도 기반 중복 공지 감지

---

### 3. 데일리 스크럼 자동 생성
**Daily Scrum Auto-Generator**

| 항목 | 내용 |
|------|------|
| 문제 | 매 영업일 아침 Confluence 데일리 스크럼 페이지를 수기로 생성 |
| 해결 | Python 스케줄러로 매일 07:00 자동 생성, 전날 "오늘 할일" → 당일 "어제 한 일" 자동 이관 |
| 기술 | Python, Confluence API, Teams Webhook |
| 성과 | 팀 전체 스크럼 페이지 수작업 100% 제거, 공휴일 자동 제외 |

**주요 기능**
- 매일 오전 07:00 자동 실행 (토/일/공휴일 제외)
- 전날 "오늘 할일" 내용(텍스트/이미지/동영상 첨부 포함) → 당일 "어제 한 일"로 자동 복사
- 생성 완료/오류/표 구조 이상 시 Teams 알림 (팀 채널/개인 채널 분리)
- 중복 생성 방지 로직

---

### 4. 빌드 머신 자산 관리 시스템
**Build Machine Asset Management System**

| 항목 | 내용 |
|------|------|
| 문제 | 빌드 머신 자산 현황 파악 불가, 추가 요청 프로세스 없음 |
| 해결 | 웹 기반 자산 관리 시스템 구축, 승인 워크플로우 + Confluence 연동 |
| 기술 | Node.js, Express, Confluence API, HTML/CSS/JS |
| 성과 | 전체 자산 통합 관리, 네트워크 모니터링, 요청-승인 프로세스 표준화 |

**주요 기능**
- 자산 현황 조회/수정/삭제, 상태별 필터링(사용중/미정/확인 필요)
- 자산 추가 요청 → 관리자 승인/거절 워크플로우
- 네트워크 & 모니터링 탭 (IP/MAC/온라인 상태)
- Confluence 버튼 매크로 연동, 표 복사 기능
- 로그인 기반 권한 관리(일반/관리자)

---

### 5. JSY 웹툴 포털
**JSY Web Tool Portal**

| 항목 | 내용 |
|------|------|
| 문제 | 내부 도구들이 분산되어 있어 접근 불편 |
| 해결 | 카드형 통합 포털 구축, 도구별 온라인 상태 실시간 표시 |
| 기술 | Node.js, Express, HTML/CSS/JS |
| 성과 | 팀 전체 도구 접근 일원화, 30초마다 온라인 상태 자동 갱신 |

**주요 기능**
- 카드형 UI로 내부 도구 통합 관리
- TCP 연결 기반 온라인/오프라인 상태 실시간 감지 (응답 시간 ms 표시)
- 로그인 기반 개인별 카드 순서 저장 (드래그앤드롭)
- 도구별 주의사항(⚠️) 표시 기능

---

## 🔧 사용 기술 스택 (Tech Stack)

| 분류 | 기술 |
|------|------|
| AI API | Claude API, Gemini API, GPT API, GitHub Copilot |
| Backend | Node.js, Express, Python |
| 협업 도구 연동 | Confluence API, Teams Webhook, Jenkins REST API |
| Frontend | HTML, CSS, JavaScript |
| 기타 | 로컬 패턴 학습, 유사도 기반 중복 감지, TCP 상태 감지 |

---

## 💡 만든 사람 (About)

게임 개발 프로젝트의 Technical PM으로서, 빌드/릴리즈 파이프라인 운영과 글로벌 일정 관리를 담당하고 있습니다.  
반복되는 수작업을 발견하면 AI API와 자동화 도구로 직접 해결하는 것을 좋아합니다.

- 빌드 리드타임 50% 단축
- 수동 업무 80% 자동화
- 180명 규모 글로벌 프로젝트 SSOT 구축 → 싱크 오류 제로화
- 글로벌 170개국 서비스 FGT 운영

> "반복되는 일은 자동화하고, 사람은 판단에 집중한다"
