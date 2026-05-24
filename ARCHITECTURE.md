# 로켓커리어연구소 — 마스터 아키텍처 인덱스

> **모든 작업 시작 전 이 파일 읽기 필수. 완료 후 반드시 업데이트.**
> 상세 구현은 각 모듈의 `CLAUDE.md` 참조.
> `admin-tool/` 기준 작업 시 ARCHITECTURE.md 경로: `../ARCHITECTURE.md`

마지막 업데이트: 2026-05-23 (sync-arch — 미문서 모듈 3종 추가, 영상 렌더 API 구현 상태 정정)

---

## 전체 모듈 인벤토리

| # | 모듈 | 상태 | 담당 에이전트 | 주요 경로 |
|---|-----|:---:|------------|---------|
| 1 | 구간3 파이프라인 | 🔒 FROZEN | — | `admin-tool/app/api/submit-diagnosis/`, `inbox/`, `send-email/` |
| 2 | 마케팅 자동화 | ✅ ACTIVE | marketing-db-agent | `admin-tool/app/api/content/`, `lib/script-generator.ts` |
| 3 | 어드민 UI | ✅ ACTIVE | admin-upload-agent | `admin-tool/app/admin/` |
| 4 | 대시보드 | ✅ ACTIVE | admin-upload-agent | `admin-tool/app/api/dashboard/` |
| 5 | 트렌드/Bitly | ✅ ACTIVE | marketing-db-agent | `admin-tool/app/api/trends/`, `admin-tool/app/api/bitly/` |
| 6 | 멘토 관리 | ✅ ACTIVE | admin-upload-agent | `admin-tool/app/api/mentors/` |
| 7 | 영상 자동생성 (Remotion Lambda) | ✅ ACTIVE | — | `admin-tool/lib/video-generator.ts`, `admin-tool/remotion/` |
| 8 | 멘토 인코딩 DB | ✅ ACTIVE | encoding-agent | `Mento_incoding_casedb_project/henry-casedb/` |
| 9 | 랜딩페이지 | ✅ ACTIVE | landing-agent | `index.html`, `style.css` |
| 10 | Phase C (Auth/포털) | 📋 PLANNED | — | `admin-tool/app/mentor/`, `admin-tool/app/mentee/` |
| 11 | 리포트/PDF/Sheets | ✅ ACTIVE | admin-upload-agent | `admin-tool/app/api/generate-pdf/`, `app/api/sheets/`, `lib/pdf.ts` |

---

## 모듈별 상세

### 🔒 모듈 1 — 구간3 파이프라인 (절대 수정 금지)

**역할:** 멘티 폼 제출 → AI 진단 생성 → 이메일 발송. 서비스의 핵심 수익 파이프라인.

**API 라우트:**
```
POST /api/submit-diagnosis     ← 멘티 폼 수신 진입점 (랜딩페이지 CTA 연결)
GET  /api/inbox                ← 진단 목록 조회
POST /api/inbox/[id]/process   ← 진단 처리
POST /api/send-email           ← Resend 이메일 발송
```

**Supabase 테이블 (스키마 변경 금지):**
```
mentees       — 멘티 기본 정보 (name, email, job_category 등)
diagnoses     — AI 진단 결과
contracts     — 멘토링 계약
session_notes — 세션 메모
mentors       — 멘토 정보
```

**핵심 파일 (수정 금지):**
```
admin-tool/app/api/submit-diagnosis/route.ts
admin-tool/app/api/inbox/ (하위 전체)
admin-tool/app/api/send-email/route.ts
admin-tool/lib/anthropic.ts   ← 기존 export 변경 금지
admin-tool/lib/supabase.ts    ← 기존 export 변경 금지 (추가는 가능)
```

> ⚠️ 이 모듈 경로 수정 요청 시 즉시 작업 중단 후 사용자 확인

---

### ✅ 모듈 2 — 마케팅 자동화

**역할:** cases_career 케이스 선별 → 4단계 AI 파이프라인 → 쇼츠+롱폼 스크립트 생성

**API 라우트:**
```
POST   /api/content/generate         ← 케이스 선별 + 4단계 스크립트 생성
GET    /api/content                  ← 큐 조회 (status: pending/approved/skipped 필터)
PATCH  /api/content/[id]             ← 수동 편집 저장
POST   /api/content/[id]/approve     ← 승인
POST   /api/content/[id]/regenerate  ← 전체 재생성
POST   /api/content/[id]/revise      ← 검수 의견 반영 수정
POST   /api/content/[id]/skip        ← 스킵 (body.reason → skip_reason 저장)
POST   /api/content/[id]/autoreview  ← 자동검수 STEP3+4 실행 (autoReview + reviseScript)
POST   /api/content/[id]/video       ← 영상 렌더 시작 → render_id 반환
GET    /api/content/[id]/video       ← 렌더 상태 조회 (?render_id=xxx)
GET    /api/content/cases            ← 큐레이션용 케이스 후보 목록
PATCH  /api/content/cases/[caseId]   ← 케이스 큐레이션 토글 (content_priority)
```

**4단계 생성 파이프라인 (`lib/script-generator.ts`):**
```
STEP 1. buildStrategy()  — 벤치마크 9항목 검토 → ContentStrategy 6항목 확정
STEP 2. buildScript()    — 전략 + 벤치마크 원본 주입 → 쇼츠+롱폼 초안
STEP 3. autoReview()     — 취준생 / 헤비마케터 / 벤치마크패턴 준수 3관점 자동검수
STEP 4. reviseScript()   — 피드백 반영 최종본
```

**Supabase 테이블:**
```
content_queue    — 스크립트 큐
  └ 주요 컬럼: id, source_case_id, week_date, priority, score
               recommended_format, title, hook, insight_lines(JSONB)
               prescription, cta, hashtags, strategy(JSONB)
               longform(JSONB), benchmark_refs(JSONB)
               review_notes, revision_count, history(JSONB)
               status(pending/approved/skipped), skip_reason, created_at

lead_analytics   — 유입 분석 (⚠️ Supabase 트리거 미실행)
bitly_clicks     — Bitly 클릭 수집
```

**핵심 파일:**
```
admin-tool/lib/script-generator.ts  ← 4단계 파이프라인 (변경 시 어드민UI 전체 영향)
admin-tool/lib/content-scorer.ts    ← cases_career 스코어링 (큐레이션 우선 → 점수순)
admin-tool/lib/market-signal.ts     ← 시장 신호 (현재 하드코딩)
```

**콘텐츠 믹스 전략:** 포괄형 2개 + 특화형 1개 세트 (세 개 단위로 생성)

**선별·생성 로직 (2026-05-23 개선):**
- `content-scorer.ts`: `content_priority`(영석님 별표) 케이스를 Claude 점수보다 우선 선별
- `selectFormat()`: 포맷을 `content_angle`(인코딩 단계 판단) 기반으로 결정, 데이터-유무는 폴백
- `caseText()`: `hook_sentence`·`content_angle`을 전 포맷 공통 주입, before/after 1200자까지 사용
- `generateScript()`/`reviseScript()`: 최근 스킵 항목(제목/훅/skip_reason)을 "반복 금지" 네거티브 예시로 주입

---

### ✅ 모듈 3 — 어드민 UI

**역할:** 전체 어드민 기능의 웹 인터페이스. 멘티 관리·콘텐츠 검수·대시보드 통합.

**페이지 목록:**
```
/admin/inbox               ← 멘티 진단 수신함
/admin/mentees             ← 멘티 관리
/admin/mentors             ← 멘토 관리
/admin/reports             ← 리포트
/admin/content/review      ← 스크립트 검토 (전략근거·검수·재생성)
/admin/content/cases       ← 케이스 큐레이션 (⭐ content_priority 토글)
/admin/content/benchmarks  ← 벤치마킹 영상 관리
/admin/dashboard           ← 유입 분석 대시보드
```

**content/review 카드 UI 구성:**
```
헤더:  N순위 / 포괄형·특화형 / 포맷 / 케이스ID / 스코어 / 생성시각
좌측:  쇼츠 — 선택된 훅 / 훅 후보 3개 / 본문 자막 / 마무리 / CTA / 해시태그
우측:  롱폼 — 제목 / 오프닝 / 섹션 아웃라인 / CTA
하단:  전략 근거(접힘/펼침) / 참고 벤치마킹 / 검수 텍스트에어리어 / 액션 버튼
```

---

### ✅ 모듈 4 — 대시보드

**역할:** 유입 경로 분석, 멘토링 전환율, 콘텐츠 성과 집계

**API 라우트:**
```
GET /api/dashboard/stats   ← 대시보드 집계 (lead_analytics 기반)
```

---

### ✅ 모듈 5 — 트렌드/Bitly

**역할:** YouTube 벤치마킹 영상 등록·분석 + Bitly 클릭 추적

**API 라우트:**
```
GET    /api/trends          ← 벤치마크 목록
POST   /api/trends          ← YouTube URL 등록 + 9항목 AI 분석
DELETE /api/trends/[id]     ← 벤치마크 삭제
GET    /api/bitly            ← 클릭 데이터 수집
```

**trend_benchmarks 9개 분석 항목:**
```
hook_type, hook_sentence, title_formula, opening_3sec,
structure_pattern, retention_technique, comment_trigger,
why_it_works, applicable_formats
```

---

### ✅ 모듈 6 — 멘토 관리

**역할:** 멘토 계정·담당 멘티 매핑 관리

**API 라우트:**
```
GET    /api/mentors          ← 멘토 목록
POST   /api/mentors          ← 멘토 생성
PATCH  /api/mentors/[id]     ← 멘토 수정
DELETE /api/mentors/[id]     ← 멘토 삭제
```

---

### ✅ 모듈 7 — 영상 자동생성 (Remotion Lambda)

**역할:** 승인된 스크립트 → AWS Lambda Remotion 렌더 → YouTube Shorts MP4 (1080×1920)

**렌더 스택:**
```
Remotion Lambda (AWS ap-northeast-2)
Pexels API  — 포맷별 배경 클립 3개 (portrait HD 우선, 랜덤 5개 중 1개)
```

**지원 포맷 5종:**
```
TIMELINE     — 커리어 여정 타임라인 (번호 뱃지)
BEFORE_AFTER — 이력서 비포/애프터 비교 (✗/✓ 컬러 분리)
CHECKLIST    — HR담당자 체크리스트
INSIDER      — 채용 내부 정보 (인용 스타일)
REVERSAL     — 반전 인사이트 (취소선 애니메이션)
```

**hook_type 3종 (훅 씬 시각 변형):**
```
숫자형 — 좌측 원형 뱃지 + # 아이콘
반전형 — 취소선 + 진실 텍스트 등장
공감형 — 대형 따옴표 배경 + 인용 스타일
```

**핵심 파일:**
```
admin-tool/lib/video-generator.ts               ← startRender(), getRenderStatus()
admin-tool/remotion/compositions/ShortsVideo.tsx ← 메인 컴포지션
admin-tool/remotion/Root.tsx                    ← Composition 등록 (id: "ShortsVideo")
admin-tool/remotion/lib/                        ← fonts.ts (브랜드 컬러), timing.ts
```

**브랜드 컬러:** `#1B2B4B` (네이비 배경) / `#27AE60` (그린 포인트)

**환경변수:**
```
REMOTION_FUNCTION_NAME
REMOTION_SERVE_URL
REMOTION_AWS_REGION   (기본: ap-northeast-2)
PEXELS_API_KEY
```

**렌더 API (`/api/content/[id]/video`):**
```
POST → startRender() 호출, render_id 반환
GET  → getRenderStatus() 조회, renderUrl(완료시) 반환
```

**미구현:**
- 어드민 UI에서 렌더 진행률 폴링 표시 (API는 구현됨, UI 연결 필요)
- YouTube 자동 업로드 (Phase 3)

---

### ✅ 모듈 8 — 멘토 인코딩 DB

**역할:** 과거 멘토링 케이스 원본 파일 → 구조화된 cases_career Supabase 레코드

**폴더 구조:**
```
henry-casedb/
├── cases/career/          ← 인코딩 완료 케이스
└── cases/_needs_review/   ← 검수 대기 (다수 잔존, 인코딩 규칙 검증 필요)
```

**cases_career 주요 컬럼 (마케팅 자동화 인터페이스 — 변경 시 즉시 통보):**
```
hook_sentence, transformation_narrative, content_angle
henry_judgment_inferred, quality_before/after
review_time_before/after, payband_signal_change
improvements[], weaknesses_before[], weaknesses_remaining[]
anonymized_before/after, job_category, career_years
outcome, outcome_days, content_priority(영석님 큐레이션 별표)
layer1~7_before/after (14개 컬럼)
```

> ⚠️ 컬럼 추가·삭제·이름변경 시 `content-scorer.ts`가 직접 참조하므로 marketing-db-agent에 즉시 전달

---

### ✅ 모듈 9 — 랜딩페이지

**역할:** 유튜브 유입 → 서비스 소개 → 진단 신청 CTA → `/api/submit-diagnosis` 연결

**배포:** `rocketcareer-landing.vercel.app`

**파일:**
```
index.html   ← 정적 단일 페이지
style.css    ← 스타일
```

---

### ✅ 모듈 11 — 리포트/PDF/Sheets

**역할:** Google Sheets에서 멘티 목록 읽기 → PDF 리포트 생성 → 어드민 다운로드 서빙

**API 라우트:**
```
GET  /api/sheets              ← Google Sheets에서 멘티 목록 조회 (getMenteeList)
POST /api/generate-pdf        ← 멘티 PDF 리포트 생성 + reports-store에 메타 저장
GET  /api/reports/[filename]  ← 생성된 리포트 파일 다운로드 서빙
```

**핵심 파일:**
```
admin-tool/lib/sheets.ts          ← Google Sheets API 클라이언트 (getMenteeList)
admin-tool/lib/pdf.ts             ← PDF 생성 (generatePdf)
admin-tool/lib/reports-store.ts   ← 리포트 메타 저장/조회 (saveReport, findReportById)
```

**어드민 UI:** `/admin/reports` ← 멘티별 리포트 생성 및 다운로드

---

### 📋 모듈 10 — Phase C (Auth/포털) — 미시작

**역할:** 단일 PIN 로그인 → 역할 기반 접근 제어 (Admin / 멘토 / 멘티 3계층)

**구현 스펙 전문:** `admin-tool/PHASE_C_TASKS_20260517.md`

**주요 변경 예정:**
```
middleware.ts          ← PIN 쿠키 → Supabase JWT 세션 교체
lib/supabase.ts        ← SSR 클라이언트 추가 (기존 export 반드시 유지)
app/login/page.tsx     ← PIN → 이메일+비밀번호
app/mentor/* (신규)    ← 멘토 포털
app/mentee/* (신규)    ← 멘티 포털
app/admin/users/ (신규) ← 계정 관리 페이지
```

> ⚠️ 시작 전 필수: mentors·mentees 테이블에 `user_id UUID` 컬럼 추가 SQL 실행

---

## 의존성 맵

```
[멘토 인코딩 DB]
      │ cases_career 레코드
      ▼
[마케팅 자동화] ──► content_queue ──► [영상 자동생성 (Remotion Lambda)]
      │                                        │
      ▼                                        ▼ (Phase 3: 미구현)
[트렌드/Bitly] ──► trend_benchmarks      YouTube 업로드
      │
      ▼
[어드민 UI] ──► /admin/content/review
[대시보드]  ──► lead_analytics (⚠️ 트리거 미실행)

[랜딩페이지] ──► /api/submit-diagnosis
                       │
                       ▼
[구간3 파이프라인] ──► mentees, diagnoses, contracts
                       │
                       ▼ (읽기전용)
               [Phase C] (멘토/멘티 포털)
[멘토 관리]  ──► mentors 테이블
```

**변경 임팩트 체크표:**
| 변경 대상 | 영향 모듈 | 확인 사항 |
|---------|---------|---------|
| cases_career 컬럼 | 마케팅 자동화 | content-scorer.ts 직접 참조 |
| content_queue 컬럼 | 어드민 UI | content/review 카드 TypeScript 타입 |
| script-generator.ts 인터페이스 | 어드민 UI, 영상생성 | API 응답 타입 일치 확인 |
| mentees/mentors 컬럼 | 구간3 🔒, Phase C | FROZEN 구간 수정 금지 |
| lib/supabase.ts | 전 모듈 | 기존 export 유지 여부 |

---

## 전체 환경변수

```
# Anthropic
CLAUDE_API_KEY

# Supabase
NEXT_PUBLIC_SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
NEXT_PUBLIC_SUPABASE_ANON_KEY    ← Phase C 시작 시 추가

# 이메일
RESEND_API_KEY
EDITOR_EMAIL

# 마케팅
BITLY_API_TOKEN

# 영상 생성
REMOTION_FUNCTION_NAME
REMOTION_SERVE_URL
REMOTION_AWS_REGION              (기본: ap-northeast-2)
PEXELS_API_KEY
```

---

## 미결 과제 트래킹

| 우선순위 | 과제 | 모듈 | 비고 |
|:-------:|-----|-----|-----|
| 🔴 높음 | lead_analytics 트리거 + content_queue.skip_reason + cases_career.content_priority 컬럼 SQL 미실행 | 대시보드·마케팅 | `supabase-marketing-setup.sql` 재실행 필요 (ALTER 재실행 안전) |
| 🟡 중간 | 렌더 상태 어드민 UI 연동 | 어드민 UI | `/api/content/[id]/video` API 구현됨, UI 폴링 화면만 연결 필요 |
| 🟡 중간 | YouTube Data API v3 연동 | 트렌드 | API 키 발급 + 검색/업로드 라우트 |
| 🟢 낮음 | Phase C (Supabase Auth 전환) | Phase C | 스펙 완성됨, 구현 미시작 |
| 🟢 낮음 | YouTube 자동 업로드 | 영상 자동생성 | Phase C 완료 후 진행 |
