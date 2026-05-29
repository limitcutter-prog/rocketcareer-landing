# 로켓커리어연구소 — 마스터 아키텍처 인덱스

> **모든 작업 시작 전 이 파일 읽기 필수. 완료 후 반드시 업데이트.**
> 상세 구현은 각 모듈의 `CLAUDE.md` 참조.
> `admin-tool/` 기준 작업 시 ARCHITECTURE.md 경로: `../ARCHITECTURE.md`

마지막 업데이트: 2026-05-30 (CaseFile 영상 **완전 로컬 렌더 전환** — `scripts/make-case.mts`+`npm run make:case` / QA 게이트(`failOnQA`) / 폰트 weight 제한 최적화 / 어드민 웹 영상 UI 제거(쇼츠 버튼·롱폼 컬럼·케이스파일 웹 버튼) / **로컬 웹 패널 `/admin/local-studio` 신설**(dev 전용 — 케이스선택→대본생성→편집→로컬렌더→미리보기))

---

## 전체 모듈 인벤토리

| # | 모듈 | 상태 | 담당 에이전트 | 주요 경로 |
|---|-----|:---:|------------|---------|
| 1 | 구간3 파이프라인 | 🔒 FROZEN | — | `admin-tool/app/api/submit-diagnosis/`, `inbox/`, `send-email/` |
| 2 | 마케팅 자동화 | ✅ ACTIVE | marketing-db-agent | `admin-tool/app/api/content/`, `lib/marketing/script-generator.ts` |
| 3 | 어드민 UI | ✅ ACTIVE | admin-upload-agent | `admin-tool/app/admin/` |
| 4 | 대시보드 | ✅ ACTIVE | admin-upload-agent | `admin-tool/app/api/dashboard/` |
| 5 | 트렌드/Bitly | ✅ ACTIVE | marketing-db-agent | `admin-tool/app/api/trends/`, `admin-tool/app/api/bitly/` |
| 6 | 멘토 관리 | ✅ ACTIVE | admin-upload-agent | `admin-tool/app/api/mentors/` |
| 7 | 영상 자동생성 (**로컬 Remotion 렌더** + TTS 나레이션) | ✅ ACTIVE | — | `admin-tool/scripts/make-case.mts`(로컬 렌더, 권장), `lib/marketing/casefile-pipeline.ts`, `tts-generator.ts`, `admin-tool/remotion/` (CaseFileVideo v7). Lambda(`video-generator.ts`)는 잔존하나 serveUrl 구형 |
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
POST   /api/content/[id]/tts         ← TTS 나레이션 생성 → audio_urls + durations 반환
POST   /api/content/[id]/casevideo   ← 케이스 → 대본→검수→TTS→조립→렌더 전 과정 자동 (오케스트레이터)
GET    /api/content/cases            ← 큐레이션용 케이스 후보 목록
PATCH  /api/content/cases/[caseId]   ← 케이스 큐레이션 토글 (content_priority)
```

**4단계 생성 파이프라인 (`lib/marketing/script-generator.ts`):**
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
admin-tool/lib/marketing/script-generator.ts  ← 4단계 파이프라인 (변경 시 어드민UI 전체 영향)
admin-tool/lib/marketing/content-scorer.ts    ← cases_career 스코어링 (큐레이션 우선 → 점수순)
admin-tool/lib/marketing/market-signal.ts     ← 시장 신호 (현재 하드코딩)
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
Pexels API     — ShortsVideo 전용 배경 클립 3개 (portrait HD, 랜덤 5개 중 1개)
브랜드 네이비  — CaseFileVideo 전용 순수 #1B2B4B 배경 (Pexels 불필요)
```

**컴포지션 2종:**

**① ShortsVideo — 5포맷 쇼츠 (기존)**
```
TIMELINE     — 커리어 여정 타임라인 (번호 뱃지)
BEFORE_AFTER — 이력서 비포/애프터 비교 (✗/✓ 컬러 분리)
CHECKLIST    — HR담당자 체크리스트
INSIDER      — 채용 내부 정보 (인용 스타일)
REVERSAL     — 반전 인사이트 (취소선 애니메이션)
```
hook_type 3종 시각 변형: 숫자형 / 반전형 / 공감형

**② CaseFileVideo — 익명 케이스 파일 시리즈 (v7, 2026-05-25 풀스크린 재설계)**
```
구조(5씬): HOOK → CASE CONTEXT → PAIR×N(BEFORE/AFTER) → CLOSING → CTA
  씬별 길이는 timing override + pair.duration으로 개별 제어(TTS 실측에 맞춤)
v7 핵심:
  - 숫자 강조: highlightMetrics() — 숫자/단위/화살표를 앰버(#FFC83D)로 자동 강조
  - BEFORE/AFTER 카드화: parseQuote()로 나레이션·인용구 분리, 빨강/초록 컨테이너
  - 초점 이동: DIFF(인사이트) 등장 시 B/A 흐려지며 위로, "⚡인사담당자 포인트" 확대
  - 풀스크린(안전영역 14~94%) + 배경 깊이(그리드/글로우/비네팅) + 하단 워터마크
  - 씬 진행 막대(SceneProgressBar)로 정지 구간 속도감 보강
```

**TTS 나레이션 (lib/marketing/tts-generator.ts) — v7 신규:**
```
프로바이더 2종:
  google(기본) — ko-KR-Chirp3-HD-Zubenelgenubi (남성, 저음 안정). 서비스계정 JWT 재사용
                 (cloud-platform scope). Chirp3-HD는 SSML/pitch 미지원 → speakingRate +
                 문장부호 리듬으로 톤 제어. 톤앤매너: 인사담당자 안정감, 하강조 평서형 통일
  polly        — Seoyeon(여성, neural). SSML prosody rate + break 지원
씬별 MP3 → Supabase Storage(tts-audio 버킷, public) → 공개 URL + 실측 duration 반환
  generateSceneTTS(id, scenes, voiceCfg) / buildTTSScenes(props) / cleanForTTS()
```

**핵심 파일:**
```
admin-tool/lib/marketing/video-generator.ts                  ← startRender(), startCaseFileRender(context/timing/audio 전달), getRenderStatus()
admin-tool/lib/marketing/tts-generator.ts                    ← generateSceneTTS(), buildTTSScenes() (Polly+Google). ⚠️ parseGooglePrivateKey() OpenSSL 3.0 호환 처리 필수 (\\n 무조건 변환·\r 제거)
admin-tool/remotion/compositions/ShortsVideo.tsx   ← ShortsVideo 컴포지션 (5포맷)
admin-tool/remotion/compositions/CaseFileVideo.tsx ← CaseFileVideo 컴포지션 (v7, 5씬)
admin-tool/remotion/Root.tsx                       ← Composition 등록 (ShortsVideo + CaseFileVideo 3종)
admin-tool/remotion/lib/                           ← fonts.ts (브랜드 컬러), timing.ts
admin-tool/scripts/regen-tts-male.mjs              ← 남성 TTS 재생성 + duration 측정 유틸
admin-tool/scripts/                               ← 개발용 유틸 스크립트 모음 (test-pipeline, test-tts, render-json, check-render 등 15개+, 배포 무관)
```

**브랜드 컬러:** `#1B2B4B`(네이비) / `#27AE60`(그린) / `#FFC83D`(앰버 — 숫자·인사이트 강조) / `#FF5C5C`(빨강 — BEFORE)

**환경변수:**
```
REMOTION_FUNCTION_NAME  (현재: ...mem3008mb-disk2048mb-600sec — 긴 영상용 600초 함수)
REMOTION_SERVE_URL (case-file-v2), REMOTION_S3_BUCKET
REMOTION_AWS_REGION (기본: ap-northeast-2)
REMOTION_AWS_ACCESS_KEY_ID / REMOTION_AWS_SECRET_ACCESS_KEY  ← Lambda + Polly TTS 공용
PEXELS_API_KEY        (ShortsVideo만 필요)
GOOGLE_SERVICE_ACCOUNT_EMAIL / GOOGLE_PRIVATE_KEY  ← Google TTS(재사용, cloud-platform scope)
NEXT_PUBLIC_SUPABASE_URL / SUPABASE_SERVICE_ROLE_KEY  ← TTS 오디오 Storage 업로드/URL
```
> ⚠️ Google TTS: GCP 프로젝트(950271061818) `texttospeech.googleapis.com` 활성화 완료(2026-05-25)
> ⚠️ Polly: remotion-user IAM에 `PollyAccess` 관리형 정책 연결 완료

**Lambda 사이트:** `case-file-v2` (S3, Audio 컴포넌트 포함)

**렌더 API (`/api/content/[id]/video`):**
```
POST (body: { type?: 'shorts' | 'case_file', context?, audio? })
  → type='shorts'    : startRender()          — insight_lines(string[]) 기반
  → type='case_file' : startCaseFileRender()  — insight_lines(BeforeAfterPair[]) + context/audio
GET  (?render_id=xxx) → getRenderStatus() 조회, outputFile(완료시) 반환
```

**TTS API (`/api/content/[id]/tts`):**
```
POST (body A: { scenes: TTSScene[], voice? } / body B: { props, voice? })
  → generateSceneTTS → { audio_urls, durations, scene_count, voice }
  voice 기본값: { provider:'google', googleVoice:'ko-KR-Chirp3-HD-Zubenelgenubi', rate:1.2 }
```

**CaseFileVideo DB 연동:**
```
insight_lines 컬럼에 BeforeAfterPair[] JSONB 배열 저장
  형태: [{ before, after, diff, duration? }, ...]
case_number: title에서 숫자 추출, 없으면 id 앞 6자리 해시
TODO: content_queue에 job_role·career_level·industry·situation 컬럼 추가 시 context 자동 구성
```

**양산 파이프라인 (판단=스킬·코드 / 실행=코드) — 오케스트레이터 완료:**
```
runCaseFilePipeline(CaseInput) — lib/marketing/casefile-pipeline.ts
  → ① generateCaseFileScript (Claude, 10룰)   케이스 → props + ttsScenes
  → ② reviewCaseFileScript   (Claude, 3관점)  검수, 실패 시 1회 재생성
  → ③ generateSceneTTS + trimSilence          나레이션 → audio + 실측 duration
  → ④ assembleProps                            duration/timing/audio 자동 주입(핸드오프)
  → ⑤ startCaseFileRender → Lambda → mp4
```
- 통합 API: `POST /api/content/[id]/casevideo`
  - body A(어드민 권장): `{ case_id, case_number?, maxPairs?, voice?, qa?, render? }` → cases_career 자동 매핑
  - body B(직접): `{ case: CaseInput, ... }`
- 스킬(Claude Code용): `.claude/commands/casefile-script.md`, `casefile-qa.md`
- 핵심 파일: `lib/marketing/casefile-script.ts`(대본+검수), `lib/marketing/casefile-pipeline.ts`(오케스트레이터), `lib/marketing/silence-trim.ts`(공백제거), `lib/marketing/casefile-input-mapper.ts`(cases_career→CaseInput), `lib/marketing/casefile-benchmark.ts`(트렌드 훅 패턴 힌트)
- 트렌드 지능 이식: `buildBenchmarkHint()`가 trend_benchmarks 훅 패턴을 "[참고 트렌드]"로 주입 (참고용, 룰 우선 안 함, 폴백). 훅 후보 `hook_candidates` 2~3개 생성(선택용). ①(script-generator)의 벤치마크 가치를 ②에 additive로 반영 — ②의 13룰 무손상
- 음성 공백제거: `@ffmpeg-installer/ffmpeg`의 `silenceremove`(threshold -50dB 필수, -38dB는 단어 붙음)
- **상세 프로세스: `admin-tool/CASEFILE_PIPELINE.md`** / 10룰·v7: `remotion/CLAUDE.md` / 톤: memory
- 🧱 **코드 경계(분리 준비, 2026-05-26)**: 무거운 마케팅 로직 11개 파일을 `admin-tool/lib/marketing/`에 격리 (`lib/marketing/README.md`). 외부 접점은 `lib/supabase`(데이터)·remotion 타입뿐. FROZEN 무관. → 향후 **marketing-engine**(별도 호스트)으로 추출 단위.
- 🧱 **잡 큐 계약**: `content_queue.casefile_status` = `queued→script→qa→tts→render→done|failed`. `enqueueCaseFileJob()` seam 준비됨(현재 미사용 — 엔진 없어 버튼은 인라인 유지). 분리 로드맵·계약: `CASEFILE_PIPELINE.md` "잡 큐 계약".

**어드민 연동 진행 상황:**
- ✅ content_queue 컬럼 추가 SQL (casefile_props/tts_scenes/audio_urls/scene_durations/voice_config/render_id/casefile_status/qa_result) → `supabase-marketing-setup.sql` (⚠️ **재실행 필요**)
- ✅ `cases_career → CaseInput` 매핑 (`lib/marketing/casefile-input-mapper.ts`) — improvements→pairs, reason·what_hr_thinks(왜 근거) 보존
- ✅ `/casevideo` body A(case_id) 자동 매핑 지원
- ✅ 파이프라인 결과 → content_queue 저장 (`lib/marketing/casefile-store.ts`, UPDATE-by-id, opts.contentId)
- ✅ 어드민 "📁 케이스파일 영상" 버튼 (`review/page.tsx` `handleGenerateCaseVideo` → `/casevideo` body C)
- ✅ 운영 노브 설정 패널 (음성 남/여·속도·pair수, localStorage 영속 → /casevideo body 전달)
- ✅ 시각 점검: 오버플로/줄깨짐 없음 (verbose 케이스 밀도만 주의)
- ✅ **렌더 상태 어드민 UI 연동 (2026-05-29)**: `render_id` content_queue DB 저장 → 페이지 새로고침 후 폴링 자동 복원(`useEffect`+`resumedRef`). `pollRender()` 헬퍼로 3곳 중복 제거. `casefileStageLabel()` 스테이지 한국어 라벨(queued→script→qa→tts→render→done).
- ✅ **승인 → 자동 ShortsVideo 렌더 (2026-05-29)**: `approve/route.ts` — `casefile_props` 없는 아이템만 `startRender()` 호출 후 `render_id` DB 저장. `maxDuration=30` 명시. CaseFile 아이템은 `/casevideo` 파이프라인 전용 유지.
- 📋 BGM 트랙 / YouTube 자동 업로드

---

### 🎚️ 모듈 7-A — 영상 생성 설정 레지스트리 (단일 진실 소스, 2026-05-25)

> **목적:** 템플릿·음성·길이·룰 등 모든 결정값을 한 곳에 박제해, 어드민/경로 변경 시 누락·충돌 방지.
> **원칙:** "룰·알고리즘은 코드 고정(품질 보증), 운영 노브만 어드민 노출". 아래 값이 곧 production 기준값.

#### (1) 템플릿
| 항목 | 값 | 출처(SoT) |
|---|---|---|
| CaseFile 컴포지션 | **CaseFileVideo v7.1** (5씬 + 글리프 안전) | `remotion/compositions/CaseFileVideo.tsx` |
| Shorts 컴포지션 | ShortsVideo (구형 5포맷) | `remotion/compositions/ShortsVideo.tsx` |
| Lambda 사이트 | `case-file-v2` (두 컴포지션 모두 포함) | env `REMOTION_SERVE_URL` |
| 동적 길이 | `CaseFileVideo` 컴포지션에 `calculateMetadata`로 props 기준 길이 계산 (없으면 기본 props 길이로 고정돼 잘림) | `Root.tsx` |
| ⚠️ 재배포 규칙 | CaseFileVideo.tsx/Root.tsx 변경 시 **반드시** `lambda sites create --site-name=case-file-v2` 재배포 | — |
| 🔴 글리프 안전 | 이모지/기호(🚀✗✓✦⭐👇⌄↓ curly“”) **금지** — Lambda Chrome엔 폰트 없어 tofu(□). 화살표=CSS삼각형, 따옴표=직선`"`. **로컬 아닌 Lambda 렌더로 검증** | `remotion/CLAUDE.md` |
| 취소선 | `textDecorationLine:'line-through'`(다줄 전체), 알파 애니메이션 | `CaseFileVideo.tsx` ExcerptCard |

#### (2) 음성 (DEFAULT_VOICE)
| 항목 | 값 | 출처(SoT) |
|---|---|---|
| provider | `google` | `lib/marketing/casefile-pipeline.ts` DEFAULT_VOICE |
| voice | `ko-KR-Chirp3-HD-Zubenelgenubi` (남, 저음 안정) | 〃 |
| rate | `1.2` (말꼬리 상승 방지) | 〃 |
| pitch | `0` (Chirp3-HD는 pitch·SSML 미지원) | 〃 |
| trim(공백제거) | `true` | 〃 / `lib/marketing/tts-generator.ts` |
| 대체(여) | Polly `Seoyeon` (provider:'polly') | `lib/marketing/tts-generator.ts` |

#### (3) 음성 공백제거 (silence-trim)
| 항목 | 값 | 출처(SoT) |
|---|---|---|
| 필터 | `silenceremove=stop_periods=-1:stop_duration=0.25:stop_threshold=-50dB:detection=peak` | `lib/marketing/silence-trim.ts` |
| ⚠️ threshold | **-50dB 고정** (-38dB는 단어 끝을 먹어 단어 붙음/어눌) | 〃 |
| 바이너리 | `@ffmpeg-installer/ffmpeg` (Remotion 번들엔 silenceremove 없음) | 〃 |

#### (4) 길이/타이밍
| 항목 | 값 | 출처(SoT) |
|---|---|---|
| duration 산식 | 씬 실측초 + 0.4s(페이드 버퍼) | `lib/marketing/casefile-pipeline.ts` `assembleProps` |
| timing.pair 기본 | pair duration 평균 | 〃 |
| 나레이션 자수 예산 | hook≤45자·context≤70자·pair≤70자(4문장)·closing≤55자·cta≤80자, **전체 45~70s** | `lib/marketing/casefile-script.ts` 룰11 |
| 맞춤법/모드 | 룰12 출력 전 오탈자 점검 / 룰13 mode(universal·specific) | 〃 |
| ⚠️ totalCaseFrames | 호출 시 **timing 인자 필수** (Lambda는 calculateMetadata가 처리) | `Root.tsx`/`CaseFileVideo.tsx` |

#### (5) 폰트·컬러 (v7)
```
TITLE 102 / HOOK 82 / BODY 56 / LABEL 38 / META 33   (CaseFileVideo.tsx)
bg #1B2B4B · green #27AE60 · ACCENT(앰버) #FFC83D · RED #FF5C5C
```

#### (6) 콘텐츠 10룰 (대본 품질)
- 코드 SoT: `lib/marketing/casefile-script.ts` SYSTEM_PROMPT (Claude 호출 시 적용)
- 문서 SoT: `remotion/CLAUDE.md` "콘텐츠 작성 룰" + `.claude/commands/casefile-script.md`
- 톤앤매너: memory `feedback_tone_voice` / `project_shorts_production`
- ⚠️ 룰 변경 시 **세 곳 동시 갱신** (코드 프롬프트 / remotion CLAUDE.md / 스킬 md)

#### (7) 렌더 옵션
```
codec h264 · imageFormat jpeg · framesPerLambda 500 · maxRetries 2 · downloadBehavior play-in-browser
Lambda 함수: remotion-render-4-0-465-mem3008mb-disk2048mb-600sec (timeout 600s)
```
출처: `lib/marketing/video-generator.ts` `startCaseFileRender` / env `REMOTION_FUNCTION_NAME`
- ⚠️ 긴/무거운 영상은 240초 함수에서 timeout → **600초 함수 필수**. framesPerLambda는 300(동시성↑ rate-limit)와 1000(청크당 timeout) 사이 **500이 균형**.
- ⚠️ 한 번에 여러 렌더 동시 발사 시 AWS 동시성 rate-limit. 운영은 순차 권장.

---

### ⚠️ 경로 정합 — 로컬 렌더 전환 (2026-05-30)

**CaseFile 영상 렌더는 이제 완전 로컬.** 어드민 웹의 영상 생성 버튼(쇼츠·케이스파일)은 모두 제거됨.

| 경로 | 트리거(현재) | 컴포지션 | 상태 |
|---|---|---|---|
| **`npm run make:case -- <caseId>`** | 로컬 CLI(`scripts/make-case.mts`) | **CaseFileVideo v7** | ✅ **권장 경로 (CLI)**. 매 실행 시 `remotion/index.ts` 즉석 번들 → 항상 최신 컴포지션 |
| **`/admin/local-studio`** (웹 패널) | 로컬 dev 서버(`app/api/local-studio/*`) | **CaseFileVideo v7** | ✅ **권장 경로 (웹)**. dev 전용(NODE_ENV=production이면 403). `script`(파이프라인 render:false)→편집→`render`(즉석 번들)→`preview`. make:case와 동일 렌더 경로 |
| `POST /casevideo` (Lambda) | (어드민 버튼 제거됨) | CaseFileVideo | ⚠️ 잔존 — 배포된 serveUrl이 **구형(ShortsVideo만)**이라 `Could not find composition CaseFileVideo` 발생. 쓰려면 serveUrl 재배포 필요 |
| `POST /video` (ShortsVideo) | (어드민 버튼 제거됨) | ShortsVideo(구형) | ⚠️ 구형 — UI 제거됨. 백엔드 라우트만 잔존 |

**🔒 정합 결정 (고정):**
1. **CaseFile 영상은 로컬에서만 생성** — CLI `npm run make:case` 또는 웹 패널 `/admin/local-studio`(dev). 둘 다 기존 TS 파이프라인(`runCaseFilePipeline`)을 그대로 재사용(SoT) → 대본·검수·TTS·조립 로직 중복 없음. 웹 패널은 대본 생성 후 hook/context/before/after/diff/closing/cta를 편집한 뒤 `/api/local-studio/render`로 즉석 번들 렌더(자막만 수정 시 음성 재생성 불필요).
2. **QA 게이트**: `runCaseFilePipeline({ failOnQA: true })`면 검수 최종 실패 시 TTS 전에 `QAGateError` throw → 불량 영상·TTS 비용 방지. 로컬 스크립트는 `--force` 없으면 게이트 ON.
3. **어드민 web 영상 버튼 제거됨** (`review/page.tsx`): 쇼츠 생성·케이스파일 영상 버튼, 롱폼 우측 컬럼, 음성·속도 노브, 케이스파일 편집 패널 모두 삭제. `content_queue` 행 생성(`/generate`)·검수/승인/스킵 백엔드는 유지.
4. **폰트 최적화**(`remotion/lib/fonts.ts`): `loadFont`에 weight(400/600/700/800/900)+subset(korean/latin) 제한 → 요청 1116→605. 잔여 CJK 슬라이스 경고는 `ignoreTooManyRequestsWarning`로 silence.

> ⚠️ Lambda 경로를 되살리려면 `CaseFileVideo` 포함된 번들을 S3에 재배포해야 함. 현재는 로컬 렌더만 정합.

---

### ✅ 모듈 8 — 멘토 인코딩 DB

**역할:** 과거 멘토링 케이스 원본 파일 → 구조화된 cases_career Supabase 레코드 + RAG 벡터 검색

**현황 (2026-05-26):** cases_career 29건 / rules_career 173건(active 168) / 임베딩 29건 완료(100%)
> 미처리 5건: case-024(JSON파싱-특수문자), case-048(크레딧소진·재시도대기), case-056·102(이미지PDF), case-084(원본없음)

**폴더 구조:**
```
henry-casedb/
├── sync_dashboard.pyw          ← GUI 현황판 (더블클릭 실행) — 로컬 vs DB 대조, 분석 실행 버튼
├── cases/career/               ← 케이스 폴더 38개 (DB 22건, 16건 처리 대기)
│   ├── case-XXX/               ← before.docx + after.docx + meta.json
│   └── _needs_review/          ← 분류 불가 케이스 검수 대기
├── output/career/analyses/     ← 분석 JSON 로컬 백업 22건
└── scripts/
    ├── [배치] batch_process.py, analyze_case.py, store_case.py, classifier.py
    ├── [동기화] sync_cases.py, watcher.py, status_check.py  ← 2026-05-16 신규
    ├── [RAG]   embed_cases.py, search_similar.py            ← 2026-05-16 신규
    ├── [운영]  weekly_routine.py                            ← 2026-05-16 신규
    └── migrations/
        ├── 001_add_is_active_to_rules.sql   ✅ 실행 완료
        └── 002_add_embedding_to_cases.sql   ✅ 실행 완료
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
embedding vector(384)  ← RAG 벡터 (paraphrase-multilingual-MiniLM-L12-v2, 2026-05-16 추가)
```

**rules_career 주요 컬럼:**
```
layer, pattern, inference, fix, frequency, case_ids[]
is_active BOOLEAN  ← 2026-05-16 추가 (케이스 삭제 시 frequency=0→FALSE 자동 처리)
```

**RAG 인프라 (2026-05-16 구축):**
```
Supabase RPC: search_similar_cases(query_embedding, match_count, job_filter, min_quality_after)
  → 코사인 유사도 top-N 케이스 반환 (ivfflat 인덱스)
연동: ver1-resume-writer SKILL.md §0 — 경력기술서 작성 전 자동 RAG 조회
```

> ⚠️ 컬럼 추가·삭제·이름변경 시 `content-scorer.ts`가 직접 참조하므로 marketing-db-agent에 즉시 전달

> 🔴 **JSONB 함정 (필독):** `improvements`·`weaknesses_before`·`weaknesses_remaining`는 Supabase가 **JSON 문자열(string)로 반환**한다 (네이티브 배열 아님). `.map()` 직접 호출 시 깨짐 → 반드시 `JSON.parse` 후 사용. `casefile-input-mapper.ts`의 `parseArr()`가 array/string 모두 방어. 신규 소비자도 동일 처리 필수.

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
POST /api/generate-report     ← 리포트 생성 (generate-pdf 연관, 별도 라우트)
GET  /api/reports/[filename]  ← 생성된 리포트 파일 다운로드 서빙
GET  /api/mentees             ← 멘티 목록 조회
PATCH/DELETE /api/mentees/[id] ← 멘티 수정/삭제
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
middleware.ts          ← 현재 HMAC 서명 토큰 → Supabase JWT 세션 교체
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

## 인증 / 보안 경계 (2026-05-24)

**역할:** 어드민 전 영역(페이지 + API) 접근 통제

**구성:**
```
middleware.ts        ← /admin/* 및 /api/* 보호. 미인증 시 API는 401 JSON, 페이지는 /login 리다이렉트
                       공개 예외: /api/submit-diagnosis(랜딩 폼), /api/auth/login
lib/auth.ts          ← Web Crypto HMAC-SHA256 세션 토큰 (Edge/Node 공용)
                       createSessionToken / verifySessionToken / verifyPin (상수시간 비교)
app/api/auth/login   ← PIN 검증 후 서명 토큰을 httpOnly 쿠키(admin_auth, 7일)로 발급
```

**쿠키 정책:** 값은 `만료시각.HMAC서명` 토큰(PIN 평문 저장 안 함), `secure`는 프로덕션만, `sameSite=lax`

> ⚠️ 새 공개 API(인증 불필요)를 추가하면 `middleware.ts`의 `PUBLIC_API` 배열에 경로를 등록해야 함
> 📋 Phase C에서 이 메커니즘을 Supabase Auth(JWT)로 교체 예정 — [모듈 10] 참조

---

## 전체 환경변수

```
# 인증 (어드민)
ADMIN_PIN                         ← 로그인 PIN (필수)
AUTH_SECRET                       ← 세션 토큰 서명 키 (선택, 미설정 시 ADMIN_PIN 사용)

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
REMOTION_FUNCTION_NAME            (현재: ...mem3008mb-disk2048mb-600sec)
REMOTION_SERVE_URL               (case-file-v2)
REMOTION_S3_BUCKET
REMOTION_AWS_REGION              (기본: ap-northeast-2)
REMOTION_AWS_ACCESS_KEY_ID       ← Lambda + Polly TTS 공용
REMOTION_AWS_SECRET_ACCESS_KEY
PEXELS_API_KEY                   (ShortsVideo만)

# TTS 나레이션
GOOGLE_SERVICE_ACCOUNT_EMAIL     ← Google TTS(남성, 재사용) — Sheets와 공용
GOOGLE_PRIVATE_KEY               ← cloud-platform scope로 TTS 호출
# (Polly TTS는 REMOTION_AWS_* 재사용 / 오디오 저장은 SUPABASE_* 재사용)
# 음성 공백제거: @ffmpeg-installer/ffmpeg (devDependency, silenceremove 필터용)
```

---

## 슬래시 커맨드 / 스킬 (`.claude/commands/`)

| 커맨드 | 역할 |
|-------|-----|
| `/sync-arch` | ARCHITECTURE.md ↔ 코드 동기화 검증 |
| `/casefile-script` | 케이스 → CaseFileVideo 대본(props+TTS) 생성 (10룰) |
| `/casefile-qa` | 생성 대본 3관점 자동 검수 (논리/HR언어/몰입) |
| `/shorts-strategy` | 쇼츠 콘텐츠 전략 (벤치마크·포맷·훅·CTA) |
| `/video-production` | 영상 자동생성 연구·결정 현황 |
| `/project-state` | 프로젝트 현황 요약 |

---

## 미결 과제 트래킹

| 우선순위 | 과제 | 모듈 | 비고 |
|:-------:|-----|-----|-----|
| 🔴 높음 | lead_analytics 트리거 + content_queue(skip_reason·**casefile 8컬럼**) + cases_career.content_priority 컬럼 SQL 미실행 | 대시보드·마케팅·영상 | `supabase-marketing-setup.sql` 재실행 필요 (ALTER IF NOT EXISTS — 재실행 안전). **casefile_props 등 8컬럼 추가됨** |
| ✅ 완료 | 어드민 CaseFile 버튼 → `POST /casevideo { case_id }` 연결 + 결과 content_queue 저장 | 어드민 UI·영상 | `review/page.tsx` handleGenerateCaseVideo + casefile-store.ts UPDATE-by-id 완료 |
| 🟢 낮음 | marketing-engine 별도 호스트 구축 (단계 2~5) | 영상 자동생성 | 코드 경계(단계 0·1) 완료. 다음: 호스팅(Railway/Render 등) 결정 → 버튼→큐 전환 → repo 분리 (`CASEFILE_PIPELINE.md` "잡 큐 계약") |
| ✅ 완료 | 렌더 상태 어드민 UI 연동 | 어드민 UI | `render_id` DB 저장·복원, `pollRender()` 헬퍼, `casefileStageLabel()` 스테이지 라벨 (2026-05-29) |
| ✅ 완료 | 어드민 승인 → 자동 ShortsVideo 렌더 트리거 | 영상 자동생성 | `approve/route.ts` — casefile_props 없는 아이템만 startRender() + render_id DB 저장 (2026-05-29) |
| 🟢 낮음 | BGM 트랙 추가 (감성 royalty-free) | 영상 자동생성 | Remotion `<Audio>` 추가 — TTS와 볼륨 믹싱 |
| 🟡 중간 | YouTube Data API v3 연동 | 트렌드 | API 키 발급 + 검색/업로드 라우트 |
| 🟢 낮음 | Phase C (Supabase Auth 전환) | Phase C | 스펙 완성됨, 구현 미시작 |
| 🟢 낮음 | YouTube 자동 업로드 | 영상 자동생성 | Phase C 완료 후 진행 |
