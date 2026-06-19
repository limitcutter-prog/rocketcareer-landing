# 작업 로그 (WORKLOG)

> /sync-arch 실행 시 자동 누적. 최신 항목이 맨 위.
> 각 블록 = 한 작업 세션. 형식: 무엇을 / 왜 / 영향. 커밋 해시는 근거.

---

## 2026-06-18 14:51 — 멘티 포털(멘티 화면) 신규 구축 — S0~S4 풀 딜리버리 (Phase A~C 코드완료·tsc 0)

**무엇을**
- **멘티 인증(매직링크·passwordless)**: `lib/auth.ts` 추가 `createMenteeToken/verifyMenteeToken`(`mentee_auth` 7일) + `createMenteeLink/verifyMenteeLink`(7일·`mlink:` 네임스페이스로 세션 오용 차단). `middleware.ts` 멘티 게이트 + `PUBLIC_API`(login/verify/logout) + matcher `/mentee/:path*`. mentees 스키마 무수정(stateless).
- **포털 UI**: `app/mentee/login`(이메일→링크) + `app/mentee/page.tsx`(여정 요약·S0~S4 로드맵·순차잠금·개인 TO-DO) + `app/mentee/forms.tsx`(회차 구조화 폼 S0 진단체크리스트/S1 Roadmap/S2 경험시트/S3·S4 업로드·파일 업로더·리포트뷰).
- **제출 파이프라인**: `/api/mentee/{login,verify,logout,me}` + `sessions/[sid]`(PATCH → `mentee_submission` JSONB + `assignment_input` 직렬화 + `assignment_done`) + `.../files`(mentee-files·`uploaded_by='mentee'`·signed URL) + `.../report(+pdf)`(sent만, `renderReportPdf` 재사용). 공용 `lib/mentee-portal.ts`(인증·소유권·노출 화이트리스트·직렬화).
- **멘토 글루**: `JourneyPanel.tsx` "멘티 초대"(매직링크 발송·링크 복사) + S1/S2 "멘티 제출됨" 배지(`aiInput`이 `assignment_input` 자동 prefill → 기존 AI 그룹화/합리화 무수정 동작). `/api/mentees/[id]/journey/invite`.
- **SQL**: `supabase-mentee-portal.sql`(case_sessions.mentee_submission/submitted_at·files.session_id — ADD IF NOT EXISTS).

**왜**
- 멘토 화면(JourneyPanel)은 있으나 멘티 셀프 제출 화면 0건. 멘토가 크몽 메시지에서 받아 손으로 붙여넣던 구조(SOP §2.3 데이터 누수 리스크)를 멘티 셀프 제출로 전환. 사장 시스템 구상(`필요작업 구조화.xlsx` "멘티로그인→관리페이지→자료 분석→TO-DO") 실현. 입력 항목 = SOP §3·운영스크립트 회차 명세 그대로.

**영향 / 후속**
- 🔒 FROZEN 무수정: mentees/diagnoses/contracts 읽기조인만. 쓰기는 case_sessions(모듈13)·files(공용)·신규 컬럼·신규 `/api/mentee/*`·lib/auth 추가분.
- 보안: 전 `/api/mentee/*` 소유권 검증(token menteeId===journey.mentee_id), `safeSession`이 judgment_note·ai_suggestion·stt_text 비노출, mentee_report는 sent만.
- 검증: `tsc --noEmit` 0 에러. **후속(사장 승인 영역)**: ① `supabase-mentee-portal.sql` 적용 ② 라이브 E2E 스모크(초대→링크→제출→JourneyPanel 자동표시→리포트) ③ 매직링크 메일 발송. ⚠️ 로컬 dev `.next` Windows 손상 이력 → 프로덕션 우회 검증 권장.
- 로드맵: 메시지(case_messages)·일정 캘린더·Phase C(Supabase Auth 승격).

**커밋**: (미커밋)

---

## 2026-06-16 22:23 — 마켓플레이스 생애주기 재설계 P1·P2 (멘토 단일화 + 로그인 통합)

**무엇을**
- **P1 멘토 단일화(#1·#2)**: `/api/mentors`(GET·POST·`[id]`)를 레거시 `mentors` → `mentor_profiles` 단일 풀로 전환. `lib/mentors.ts` 정규화(`expertise`→specialties·`headline`→소속·`approved`→active). 멘토 심사 승인분이 멘토 관리·진단 멘토지정 드롭다운에 즉시 노출. `supabase-mentors-unify.sql`(레거시 1행 보존, 선택).
- **P2 로그인·권한 통합(#3)**: 통합 `/login`(사장 PIN·스태프/멘토 email+pw 단일 진입·역할별 라우팅). `/staff-login`·`/mentor/login`→`/login` 리다이렉트, 로그아웃·미들웨어 멘토 리다이렉트도 `/login`. `/api/auth/me` 멘토 역할 추가 + `PUBLIC_API` 공개(역할만 반환).

**왜**
- 사장 보고 4건의 근본이 멘토·멘티·인증 데이터 파편화. mentor_profiles/diagnosis 단일화로 #1·#2·#3 해소. 확정안: mentor_profiles 단일화 + 모든 신청 진단 경유.

**영향 / 후속**
- 사장 PIN·미들웨어 owner 분기 무변경 → 잠금0(프로덕션 사장 로그인 200·me=owner 확인). 멘티 mentor_id 배정 0건이라 재매핑 불필요.
- 검증: P1 프로덕션 /api/mentors 1→7명 전환. P2 로컬 E2E(사장·스태프·멘토 역할·잠금0) + 프로덕션 사장 무손상. tsc 0.
- ⚠️ 로컬 dev `.next` Windows 손상 반복 → 실데이터·프로덕션으로 검증 우회.
- **후속**: P3(쇼케이스 개별신청→진단 경유→mentees 생성→멘토 수락 시 case_journey 자동생성·배정)·P4(case 메시지 스레드+파일 교환) 미착수 — 다음 집중 세션.

**커밋**: `admin-tool` `fdfcad3`(P1)·`9ebd99e`(P2) / `landing` (없음)

---

## 2026-06-14 23:18 — RBAC 권한 3층 + 보안 강화 + 멘토 가입 동의 + 진단/리포트 개선 (프로덕션 배포)

**무엇을**
- **#6 RBAC 3층**: 사장(PIN·무변경)/운영스태프(`staff_users`·`/staff-login`·`staff_auth`)/멘토. `middleware` 역할분기(owner→전체 / staff→`lib/permissions` OWNER_ONLY 외 허용 / 미인증 차단), 네비 역할필터, `/admin/staff`·`/api/staff`(+`[id]`)·`/api/auth/{staff-login,staff-logout,me}`.
- **보안**: 전 public 테이블 RLS on(`supabase-enable-rls.sql`) + `mentee-files` 버킷 private + STT 녹취 경로저장(public URL 제거). anon 키 코드 노출 없음(grep) 확인.
- **멘토 가입 동의**: `/showcase/apply`에 PII 수집·이용 동의 필수(`mentor_applications.consent_*`, `supabase-mentor-consent.sql`). 동의 미전송으로 막혔던 신청 정상화. 초대링크 `/showcase/apply` 통일·`/join-mentor` 제거.
- **진단 리포트**: AI 추천상품 STEP4 자동체크 + 선택 반영 재생성(컨텍스트 주입), 추천서비스 박스 제거(/report + 🔒메일 사장승인), /report 카카오 문의 버튼.
- **멘토링 리포트 메일** 카드 레이아웃 재설계 + 카카오(이메일·PDF). **조직 콘솔** 협업 R/R 로그 표시·즉시반영. **랜딩** 멘토 매칭 보조 링크.

**왜**
- 멘토 확장 대비 권한 분리(나=전체 / 스태프=제한). 보안 경고(RLS off·public 버킷) 시정. 진단 리포트 추천 중첩·멘토 가입 동의 부재(법적) 해소.

**영향 / 후속**
- 🔒 FROZEN: send-email 박스만 사장승인 제거(로직 무변경). RLS는 service_role 우회로 앱 무중단(검증: mentees·diagnoses 읽힘).
- E2E 전부 통과: 사장 전체200·잠금0(프로덕션 로그인 200 확인) / 스태프 허용200·사장전용307·API403 / 멘토 동의 제출 성공·DB저장 / 리포트 박스제거·카카오 / 조직 협업 반영. tsc 0.
- 배포 Ready 확인: admin-tool 3건 + landing 1건 (Vercel Production).
- 후속: 운영스태프 메뉴 경계 미세조정, `/api/mentors` 메서드별 권한, 녹취 재생 필요 시 서명 URL.

**커밋**: `admin-tool` `54de08e`·`b243e0e`·`1f397a3` / `landing` `488cab1`

---

## 2026-06-14 — 모듈13 STT 파이프라인 구현 (2단계 완성)

**무엇을**
- `lib/mentoring/stt.ts` 신규: Google STT V1 REST + ffmpeg 5분 청크 분할. JWT 인증은 tts-generator.ts 패턴 그대로 재사용. 1시간 음성 → 12청크 × 순차 STT → 텍스트 합산.
- `stt/route.ts` 신규: PATCH(Track A 텍스트 직접 저장) · POST(Track B 파일 업로드→Supabase Storage→STT→DB). maxDuration=300.
- `report-generator.ts` 수정: sttText 6번째 인자 추가, `[녹취 STT 내용]` 프롬프트 주입, max_tokens STT 있으면 5000.
- `report/route.ts` 수정: stt_text SELECT + generateMenteeReport 6번째 인자 전달.
- `JourneyPanel.tsx` 수정: Session 인터페이스 recording_url/stt_text 추가, STT 섹션 UI(Track A textarea+저장 / Track B 파일업로드+상태), genReport 버튼 "STT 포함" 배지.

**왜**
- SOP §2.4 자동화 1순위: 녹취→STT→리포트. 1시간 파일은 Google STT 10MB 한계 초과 → ffmpeg 5분 청크로 우회.

**영향 / 후속**
- 모듈13 2단계 코드 완성. 실사용 전 Google STT API(speech.googleapis.com) GCP 활성화 확인 필요.
- 3단계(멘토 포털 운영 도구) 미착수.

**커밋**: `admin-tool` (미커밋) / `landing` (미커밋)

---

## 2026-06-13 22:07 — 운영 콘솔 prod 가드 + 모듈13·14 프로덕션 배포

**무엇을**
- 운영 콘솔 prod 숨김 가드: `app/admin/layout.tsx` 네비 `devOnly` 필터(prod 제외) + `app/admin/org/page.tsx` prod 시 "로컬 dev 전용" 안내(hooks 분리 래퍼 `OrgConsole`).
- 살아난 dev 서버(3000)로 콘솔 풀스택 검증: 로그인 200 · `/api/org/overview`(8부서·5엣지·목표 3섹션) · agents 6섹션 · `/admin/org` 200.
- 커밋 `5d3e1d0`(admin-tool, 40파일) → `git push origin HEAD:main` FF → Vercel Production 빌드 **● Ready**(3m, `admin-tool-c91ikgw46`) 확인.

**왜**
- 사장 지시 "배포해, 로컬 전용은 숨겨두고". 콘솔은 dev 전용(로컬 파일 fs)이라 prod 숨김, 모듈13(딜리버리)은 내부 어드민으로 라이브.

**영향 / 후속**
- 배포 범위: 모듈13(멘토링 딜리버리·리포트) + 모듈14(콘솔, prod 숨김) + casefile 수정(`a33ec1c`). 전부 admin 인증 뒤·고객 노출 0·🔒 FROZEN 무수정. PII·대용량 미포함(점검).
- 콘솔 prod 3중 차단: 네비 숨김 + 페이지 안내 + API 403. 데이터는 로컬 `org/*.md`에만.
- 루트 거버넌스 파일(org/·ARCHITECTURE·WORKLOG·랜딩)은 미커밋(배포 비대상, 루트 대용량/PII 혼재로 선별 커밋 필요).
- 후속: 모듈11 `lib/pdf.ts` 복구(L-0002, 별개) · 콘솔 Phase 2(Supabase 이전 시 prod 가동).

**커밋**: `admin-tool` `5d3e1d0` (→main 배포·Ready) / `landing` (미커밋)

---

## 2026-06-13 21:45 — 회사 운영 콕핏(org 콘솔) 구축 — 모듈14 GUI 레이어

**무엇을**
- 데이터 계층 `admin-tool/lib/org/` 7파일: `registry`(부서 8+고위험키워드)·`markdown`(섹션/표 파서)·`ledger`(append-only r/w)·`agents`(frontmatter 보존)·`objectives`·`state`(읽기)·`orgchart`(CHARTER §6 페어링·R/R append).
- API `admin-tool/app/api/org/` 7라우트(전부 prod 403 가드): `overview`·`agents/[name]`(GET·PATCH)·`ledger`(GET·POST)·`ledger/[id]`(PATCH)·`objectives`(GET·PUT)·`state/[dept]`·`orgchart`(GET·PATCH).
- UI `admin-tool/app/admin/org/page.tsx`: 3탭(조직 맵=계층0~3 노드·검수엣지 / 오더 보드=상태 칸반·드래그·새 오더 / 목표) + 부서 드로어(설정=직무·⚙️판단로직 / 오더 / 현황 / 협업). 어드민 NAV에 `조직 운영` 추가.
- 검증 스크립트 `scripts/verify-org.mts` — 실제 org/ 파일 read/write 라운드트립.

**왜**
- 사장 요청: 조직 관계·각 부서 오더와 답·부서간 커뮤니케이션→결과를 보기 좋게 보고, 거기서 오더·직무범위·판단로직·협업을 직접 조정하는 '회사 운영 장치'. 데이터는 이미 org/*.md에 구조화돼 있어 GUI 한 겹만 추가.
- 확정: 분리형(설정=대시보드/실행=Claude Code) + 로컬 dev 우선(파일=SoT, Supabase 불필요).

**영향 / 후속**
- 🔒 FROZEN·admin-tool 코드·Supabase 미접촉. 쓰기는 org/+경영 8 에이전트만. LEDGER append-only, 고위험(CHARTER §4) 오더 자동 승인대기.
- 검증: `verify-org.mts` 통과 — LEDGER 5건 파싱·append L-0006(5→6)·patch 반려·타행보존·원복, 에이전트 no-op 라운드트립 **바이트 동일(무손실)**, 고위험 게이트(가격/배포=true, 검수=false), CHARTER §6 6엣지. tsc 0.
- ⚠️ 브라우저 E2E 보류: 3000 dev 서버를 타 세션(L-0002 lib/pdf.ts)이 점유·500 상태 → 데이터 계층은 스크립트로 검증. 서버 가용 시 `/admin/org` 스모크·prod 403 확인 권장.
- 후속(Phase 2 배포): org 데이터 Supabase 이전 + `/org-sync` 브리지.

**커밋**: `admin-tool` (미커밋) / `landing` (미커밋)

---

## 2026-06-13 16:56 — 리포트 버그픽스 + 표현 개선 (S2 JSON 잘림·가운뎃점·로드맵 레이블)

**무엇을**
- `report-generator.ts`: `repairTruncatedJson` 추가. `parseJson` 개선(repair 폴백 + `·`→`, ` 후처리). S2 max_tokens 4000→6000.
- `report-pdf.tsx` + `report-generator.ts`: 로드맵 레이블 변경 — S2=경험 구체화, S3=심화과정, S4=심화과정(추가).

**왜**
- S2 리포트가 경험×그룹 전체 커버로 출력이 길어지면서 max_tokens 4000에서 JSON 잘림 → 502 발생. experience-suggest.ts의 동일 해결 패턴 적용.
- 가운뎃점(·) 연결 표현이 리포트 가독성 저하 — 쉼표+공백으로 통일.
- 로드맵 표시명이 내부 구현 용어("경험 디깅", "기본형 문서")라 멘티 전달 관점에 안 맞음.

**영향 / 후속**
- S2 리포트 정상 생성 확인(사용자 검증). tsc 에러 0.
- ARCHITECTURE.md·PLAN.md 레이블 동기화 완료.

**커밋**: `admin-tool` (미커밋)

---

## 2026-06-13 — AI 운영체계(모듈14) 구축 + SQL 마이그레이션 완료 + 퍼널 측정 체계 수립

**무엇을**
- 모듈14 AI 자율 협업 운영체계 구축: `org/`(PROTOCOL·IMPACT_MAP·CHARTER·OBJECTIVES·LEDGER·ORG_CHART·state 6종) + 에이전트 8개(커리어랩장·분신·6본부장) + 슬래시커맨드 4개(`/order`·`/standup`·`/strategy-review`·`/org-audit`)
- L-0001 SQL 마이그레이션 검증 완료: case_journeys·case_sessions·lead_analytics·bitly_clicks·trend_benchmarks·content_priority 생성 (PostgREST 200 확인)
- lead_analytics 트리거 LIVE: `EXCEPTION WHEN OTHERS THEN NULL` 하드닝 적용 — 진단 접수(FROZEN diagnoses) 롤백 불가 보장. 트랜잭션 테스트 통과.
- 랜딩페이지 v2 수정: `mode: 'no-cors'` 제거, `concern` 필드명 수정(`주요고민`→`concern`), URL 파라미터 추적 추가(`?ref=youtube_shorts` 등 → referral 자동 주입), 서버 에러 감지 가능
- Bitly API 토큰 등록 완료, 대시보드 동기화 버튼 정상화

**왜**
- 1인 운영 → AI 에이전트 자율 협업 체계로 전환(오더 인테이크·부처별 현황·객관성 검증). 매출·품질 목표(OBJECTIVES.md) 기준 전 본부 유동 재정렬.
- SQL 미실행으로 모듈13 회차 API 500, 마케팅 퍼널 측정 불가 상태였음 → 해소.
- 랜딩페이지 `no-cors`로 서버 에러 감지 불가 + concern 필드 누락 → 데이터 정합성 복구.

**영향 / 후속**
- 신규 진단 신청부터 lead_analytics 자동 수집 시작(2급 지표 확보)
- 대시보드 `채널별 유입` 실데이터 쌓히기 시작
- 후속: L-0002(lib/pdf.ts 복구, 타 세션 진행 중), L-0003(퍼널 측정 운영), Bitly 링크에 `?ref=youtube_shorts` 파라미터 추가 필요(영석님 직접)

**커밋**: `admin-tool` (미커밋) / `landing` `a5d8a0f` (루트, 마지막 sync)

---

## 2026-06-13 — S1/S2 리포트 구조 재설계 + RESEND 도메인 반영

**무엇을**
- `report-generator.ts`: `S1GroupCard`·`S2ExperienceMapping`·`RoadmapInfo` 인터페이스 신규 추가. S1 전용 `group_cards[]`(fit_summary·appeal_points·approach_tips·hr_read), S2 전용 `experience_matrix[]`(experience × per_group[positioning·key_message]), 전 회차 `roadmap`(step 1~5, 코드 삽입). `buildUserContent()` 분기로 세션별 프롬프트 분리.
- `report/route.ts`: S2 생성 시 S1 `ai_suggestion` 자동조회 → `s1Context` 전달 (그룹명 참조로 포지셔닝 매트릭스 품질 향상).
- `report-pdf.tsx`: `Roadmap`(5단계 인디케이터)·`GroupCards`(네이비 헤더 + 섹션별 구조)·`ExperienceMatrix`(경험 헤더 + 그룹행) 컴포넌트 추가. `[style, active && style2]` → 삼항 연산자 수정(react-pdf TS 호환).
- `JourneyPanel.tsx`: UI 미리보기에 로드맵·그룹카드·매트릭스 동기화.
- `.env.local`: `RESEND_FROM_EMAIL=noreply@rocketcareer.co.kr` (도메인 검증 완료).
- `tsc --noEmit` 에러 0 검증.

**왜**
- 사용자 피드백: S1은 단순 줄글이 아닌 그룹별 카테고라이즈된 정보 필요. S2는 동일 경험이 그룹마다 다르게 어필되는 매트릭스 형태 필요. 발신 주소 `onboarding@resend.dev` → 브랜딩 개선.

**영향 / 후속**
- 🔒 FROZEN 무수정. 기존 리포트 구조(highlights) 호환 유지(S1 없으면 highlights 렌더).
- 기존에 생성된 `mentee_report` JSONB는 group_cards/experience_matrix 없이 저장됐으므로 재생성 권장(재생성 버튼 제공 중).
- 후속: 녹취→STT→8섹션 리포트(2단계 잔여, SOP 자동화 1순위).

**커밋**: `admin-tool` (미커밋) / `landing` (이 sync 커밋)

---

## 2026-06-13 01:17 — S2 경험 합리화 AI + 환각·파싱·OOM 해결

**무엇을**
- S2 경험 디깅 AI: `experience-suggest.ts`(합리화 번역·과대표현 정직화·리스크(종교 등)·대표경험) + `suggest-experience` API(**S1 그룹 자동참조**) + S2 카드 UI. `ai_suggestion` 컬럼 공용(GroupSuggestion|ExperienceSuggestion union).
- 환각 차단: 입력 줄단위 `ref` 매핑 + `experience` **원문 강제 교체(코드)** → 창작/누락 0(20항목 검증).
- 파싱 잘림 해결: `max_tokens` 8000 + 출력 압축(experience 모델 생성 생략) + `repairTruncatedJson`(잘림 복구 → 502 방지).
- dev OOM("인터널 오류") 해결: `scripts/dev.mjs` 래퍼(NODE_OPTIONS=8192) → `package.json` dev 교체. cross-env는 네트워크 차단 미설치라 의존성 0 node 래퍼.

**왜**
- "S2 개선" 피드백: AI 결과 신뢰도(창작 제거)·실사용 안정성(긴 입력 502·dev 인터널오류=OOM) 확보.

**영향 / 후속**
- 🔒 FROZEN 무수정. tsx 직접 + dev API E2E 검증(20항목 502 없음·~48s).
- **후속(사용자 신규 요청)**: S1/S2 결과를 **멘티 전달용 리포트**로 가공(직무·맥락·임팩트·세부설명) + **PDF UI/UX·Export·메일 발송** → 모듈13 2단계(`lib/pdf.ts` throw 복구 선행).

**커밋**: `admin-tool` (미커밋) / `landing` (해당 없음)

---

## 2026-06-11 23:32 — 모듈13 멘토링 운영 1단계 + S1 AI 그룹화 (신규 딜리버리 축)

**무엇을**
- 모듈13(멘토링 딜리버리) 신규 등재 → 1단계: `case_journeys`·`case_sessions` 테이블, journey/sessions API, `JourneyPanel.tsx`(회차 S0~S4 운영·과제 게이트·재량 기록·이용기간 OT+3개월·트랙·긴급/휴면).
- 피드백 반영: ① `makeup` 계약 있을 때만 노출 · ③ 회차 순차 잠금(이전 `done` 전까지 🔒).
- AI 보조 레이어 첫 구현 — **S1 그룹화**: `lib/mentoring/group-suggest.ts`(Claude sonnet-4-6) + `suggest-groups` API + S1 카드 UI. `case_sessions.ai_suggestion`(JSONB)·`assignment_input`(TEXT) 추가.
- E2E 검증: 여정 생성·회차 PATCH·실 Claude 그룹화·DB 저장. SOP S1 재현(직무테마 묶기·"생산관리 vs 공정품질" 교정·커리어자산 제외).

**왜**
- SOP v1.1·운영스크립트가 정의한 "결제 후 5세션 실행" 딜리버리 축이 어드민에 비어 있었음(녹취→리포트 코드 0건). `makeup`("회사선택→이력서 Make-up")=SOP 5세션 매핑. FROZEN 우회로 신규 `case_*` 테이블 분리.

**영향 / 후속**
- 🔒 FROZEN(mentees/contracts/session_notes) 무수정·FK 읽기조인만. SQL 2회 실행됨(테이블 + AI컬럼).
- 후속: S1 공고명 보존 6/10 튜닝 · S2 경험디깅 AI(진행 중) · 2단계 녹취→STT→리포트(`lib/pdf.ts` throw 복구 선행).
- 미커밋(admin-tool 통째 untracked) · 검증용 테스트 여정(멘티 39a678f1) 잔존.

**커밋**: `admin-tool` (미커밋) / `landing` (해당 없음)

---

## 2026-06-07 — 멘토링 마켓플레이스(모듈 12) 프로덕션 배포

**무엇을**
- `feature/marketplace-phase1` → main **FF 머지·푸시**(admin-tool `0abd512`) → Vercel 자동배포. 마켓플레이스 3커밋(쇼케이스·지원/심사·멘토포털) + 영상/IMC/BGM/인용구 개선 13커밋 = 16커밋 라이브.
- 신규 **SQL 4종**(setup→apply→auth→seed) Supabase 실행 완료 — 11테이블·멘토 로그인 컬럼·시드(영석님 1인+서비스2+후기·피드).
- 배포 전 점검: middleware PUBLIC_API(showcase·mentor login/logout) 등록 확인, 공개 read API 4종 graceful(테이블 없어도 무중단), 인증 플로우(지원→어드민 승인 UI→PBKDF2 로그인) 완결, `tsc --noEmit` 통과.

**왜**
- 모듈 12가 브랜치에만 있어 "BUILT·미배포"로 죽어있던 양면 마켓플레이스를 실서비스화(사용자 "지금 라이브" 결정).

**영향 / 후속**
- 모듈 12: 🚧 BUILT·미배포 → ✅ ACTIVE. 미결 'SQL'·'머지/배포' 해소. IMC v2 §1 'Track B 미운영' → 'Phase 1 라이브' 정합.
- ⚠️ main 직접 푸시가 auto모드 가드레일에 차단 → 사용자가 `Bash(git push:*)` 권한 부여 후 **단독** `git push`로 진행(복합/`git -C`는 룰 미매칭).
- 🟡 후속: ① Vercel 빌드·`/showcase` 스모크 테스트 ② 멘토 인증 이원화(password_hash↔모듈10 Supabase Auth) 통합 ③ 결제 Phase 3 ④ Bitly 트래킹 유료 이슈로 보류(무료 대안=lead_analytics UTM, 별도 스코핑).
- `admin-tool/CLAUDE.md` showcase/mentor 행 '배포' 갱신(미커밋).

**커밋**: `admin-tool` `0abd512`(→main 배포) / `landing` (이 sync 커밋)

---

## 2026-06-06 19:56 — 인용구 과길게 추출 문제 (발췌 제한 + 렌더 클램프)

**무엇을**
- BEFORE/AFTER 인용구가 원문 문단을 통째로 가져와 카드를 넘치던 문제. **2단 방어**로 시정.
- ① 생성: before = 약점 핵심 구절만 발췌(≤45자, 원문 표현 유지, 길면 '…'), after 명사형 ≤40자, diff ≤35자. **문단 통째 인용 금지.** (`casefile-script.ts` SYSTEM_PROMPT 룰2b + `generateSubtitlesFromNarration`)
- ② 렌더: ExcerptCard 인용구 `-webkit-line-clamp:4`(기존 autofit·overflowWrap에 더해 어떤 데이터도 카드 내 고정).
- 룰 4곳 동기화: casefile-script.ts / remotion CLAUDE.md / casefile-script.md / casefile-qa.md.

**왜**
- before 원문(cases_career improvements[].before_sentence)이 장문 run-on이면 그대로 들어가 카드 깨짐. 직전 wrap 수정(23:39)은 잘림만 막았고, 텍스트量 자체가 과함 → 추출 단계에서 발췌.

**영향 / 후속**
- 신규 생성분부터 ≤45자 발췌로 ~2줄. 기존 장문 데이터도 렌더 클램프로 방어. 렌더·구조 무관.

**커밋**: `admin-tool` 0abd512 / `landing` (이 sync 커밋)

---

## 2026-06-05 23:43 — /sync-arch: IMC 정합 표류 시정 (브랜드 카드 오프닝→클로징)

**무엇을**
- IMC 정합 체크에서 표류 발견·시정: 브랜드 카드를 오프닝→클로징으로 옮겼는데(코드 c877694) **IMC v2 §1·구조·Phase0가 아직 "맨 앞 인트로·5.0s"** 로 남아있었음.
- IMC v2 §1(브랜드 카드 = 클로징·4.4s), v1→v2 표, 5씬 구조(맨 뒤 브랜드 카드), Phase0(+BGM) 정합. ARCHITECTURE "📣 마케팅 전략 레이어" 표 행도 "클로징 아웃트로·CTA 뒤"로 갱신.

**왜**
- 코드는 바꿨는데 상위 전략 문서(IMC)가 안 따라와 표류. `/sync-arch` 마케팅 정합 절차가 이를 잡아냄(코드↔전략 동기화 목적 달성).

**영향 / 후속**
- 경로 7개 전부 존재. 화법·훅·CTA 룰 4곳 동기화 상태 유지(이전 세션 반영분).

**커밋**: `landing` (이 커밋 — ARCHITECTURE·WORKLOG·IMC v2)

---

## 2026-06-05 23:39 — BEFORE/AFTER 인용구 텍스트 넘침(잘림) 수정

**무엇을**
- CaseFileVideo 인용구가 가로로 넘쳐 잘리던 버그 수정. 원인: 전 텍스트가 `wordBreak:'keep-all'` 단독이라 **공백 적은 긴 한글 원문이 줄바꿈 안 됨**.
- ExcerptCard 인용구: 길이별 폰트 자동 축소(`fitQuoteSize` 56→48→42→36→32) + `overflowWrap:'anywhere'`. hook·diff·closing에도 `overflowWrap:'anywhere'` 추가.

**왜**
- BEFORE 원문(cases_career 실제 약점, 길이 통제 불가)이 길면 카드를 넘어 화면 밖으로 잘림 → 가독성·완성도 훼손.

**영향 / 후속**
- 렌더 로직만 변경(대본·구조 무관). 검증: 긴 무공백 BEFORE 원문 테스트 props로 2줄 wrap+축소 확인.
- 후속(선택): 매우 긴 원문은 대본 단계에서 요약하는 룰도 고려 가능(현재는 렌더가 방어).

**커밋**: `admin-tool` 72d19e2 / `landing` (이 sync 커밋)

---

## 2026-06-05 23:27 — 브랜드 카드 오프닝→클로징 이동

**무엇을**
- 브랜드 발사 카드(4.4s)를 **영상 맨 앞(오프닝)에서 맨 뒤(클로징 아웃트로)로 이동**. 시작은 기존대로 HOOK부터(오프닝 없음).
- 메인 컴포지션: 씬 시작점 INTRO_F 오프셋 제거(hookStart=0…), CTA 뒤에 브랜드 카드 Sequence(`outroStart`). 끝은 로고 락업으로 마무리(브랜드 카드 끝 페이드아웃 제거, CTA→브랜드 페이드인).
- BGM 볼륨 재배치: 본문 0.05 / CTA 0.045 / **아웃트로 0.11**(끝 구간만 음악 허용). totalCaseFrames 동일(위치만 이동).

**왜**
- 오프닝 4.4s가 쇼츠 **초반 집중도(리텐션)를 크게 깎음**. 첫 1초가 생명인 쇼츠라 훅을 맨 앞, 브랜드는 잔상으로 남기는 클로징이 유리.

**영향 / 후속**
- 영상 총길이 동일(브랜드 카드 위치만 이동). 다음 렌더부터 적용.
- 검증: 시작 프레임=HOOK, 끝 프레임=브랜드 카드 스틸 확인. 전체 렌더 정상.

**커밋**: `admin-tool` (이 커밋) / `landing` (이 sync 커밋)

---

## 2026-06-05 23:26 — 멘토링 마켓플레이스 모듈 12 거버넌스 등재 (표류 시정)

**무엇을**
- 그간 ARCHITECTURE.md에 **전혀 없던** 양면 마켓플레이스를 **모듈 12**로 등재: 인벤토리 행 + 상세 섹션(페이지 7·API 11·신규 테이블 11) + 미결과제 3건 + 헤더.
- 발견한 **거버넌스 모순** 명시: ① IMC v2 §1 "Track B(모두의커리어) 미운영" ↔ 실제 Phase 1 구현됨 ② 모듈 10 "멘토 포털 미시작" ↔ `/mentor` 포털·멘토 로그인 이미 구현(단 Supabase Auth 아닌 자체 `password_hash`).
- `admin-tool` `feature/marketplace-phase1` 14커밋 **원격 백업 푸시**(그간 로컬 전용 → GitHub). main 미머지=미배포 상태 유지.

**왜**
- 사용자 "놓치고 있는 것" 점검 중, 핵심 작업이 **푸시 안 된 브랜치**에만 있어 `/sync-arch`가 한 번도 못 본 거버넌스 맹점 발견. 백업+등재로 가시화.

**영향 / 후속**
- 🔴 마켓플레이스 SQL 4종(setup→apply→auth→seed) 미실행 → `/showcase`·`/mentor` 동작 불가. Supabase 수동 실행 필요.
- 🟡 main 머지·배포 결정 + `/api/showcase/*` PUBLIC_API 등록 점검(미등록 시 401).
- 🟡 IMC v2 §1 갱신 + 멘토 인증 이원화(모듈 10·12) 통합 방향 결정.
- ⚠️ 별건: 인코딩 DB(1.8GB·실제 이력서 PII) 백업 0 — git 부적합, 비공개 클라우드 백업 별도 결정.

**커밋**: `admin-tool` fd8b3fe·ea6dc7e·544151e 외 (마켓플레이스 구현, 이번에 원격 백업) / `landing` (이 sync 커밋 — ARCHITECTURE·WORKLOG·admin-tool CLAUDE.md)

---

## 2026-06-03 20:33 — IMC v2 정합 + 거버넌스에 마케팅 전략 레이어 박제

**무엇을**
- IMC 기획안 **v2**(`marketing_division/..._v2.md`)를 sync-arch 누적 결정에 정합: BI 컬러 실제 운영 체계 통일(골드크림 폐기), 발사·런칭 메타포 전면 통일, 자동 양산 파이프라인·CaseFileVideo·인트로 V5·화법룰·업로드 키트 반영. (인트로 4.4s·상단 헤더·§12 CTA/화법은 17:03·17:06 세션이 코드 정합 완료 → 문서 일치 확인)
- **ARCHITECTURE.md에 "📣 마케팅 전략 레이어(IMC)" 섹션 신설**: IMC 기획안 v2를 전략 SoT로 등재 + 전략결정→코드 매핑 표 + 정합 규칙. 헤더·슬래시커맨드 표 갱신.
- **`/sync-arch` 스킬 확장**: 경로 확인에 `marketing_division/` 추가 + "마케팅 정합 절차(IMC↔코드)" 신설 + 출력 형식에 "📣 마케팅 정합(IMC)" 라인 추가.

**왜**
- 그간 sync-arch는 코드(`lib/marketing` 등)만 검증하고 상위 마케팅 전략(IMC)은 거버넌스 밖이라, 코드·전략이 따로 노는 표류를 못 잡았다. 사용자: "아키텍처에 마케팅도 중요" → 전략 문서를 코드와 동급 동기화 대상화.

**영향 / 후속**
- 이후 IMC 결정 변경 시 `/sync-arch`가 코드와의 정합을 점검·리포트(양방향 표류 방지).
- 다음 `/sync-arch` 실행부터 결과에 "📣 마케팅 정합(IMC)" 라인이 포함된다.

**커밋**: `landing` (이 sync 커밋) — ARCHITECTURE.md·WORKLOG.md·.claude/commands/sync-arch.md

---

## 2026-06-03 17:30 — CaseFileVideo BGM 추가 (TTS 우선·저볼륨)

**무엇을**
- 배경음악 도입. `remotion/lib/bgm.ts` `BGM_CONFIG`(SoT) + CaseFileVideo 전체 1개 루프 `<Audio>` + 프레임 볼륨.
- 볼륨: **인트로 0.11 → 본문 0.05 → CTA 0.045(억제)** + 시작/끝 페이드, 인트로→본문 하강 램프. **TTS 절대 우선**(본문 0.05로 TTS 1.0 아래).
- props `bgm`: false 끔 / 문자열 src 교체 / 미지정 defaultSrc. `bundle({publicDir})` 추가(render 라우트·make-case, staticFile 해석).
- 음원: Pixabay "Medical Doctor Clinic Background"(prettyjohn1, Corporate). **코드만 커밋, mp3는 .gitignore로 제외**(저작권 확인 전 바이너리 박제 보류, `public/bgm/` 로컬만).

**왜**
- 본문이 음성만 있어 어색 → 교육형·HR 분석형에 어울리는 코퍼레이트 BGM으로 분위기 보강. 단 TTS 가청성 최우선.

**영향 / 후속**
- 저작권: **일부공개 업로드 후 제한 여부 확인 → 뜨면 `BGM_CONFIG.enabled=false`로 즉시 제거.**
- mp3 미추적이라 다른 환경/클린클론에선 `public/bgm/clinic-corporate.mp3` 수동 배치 필요(없으면 staticFile 404 → BGM만 빠지고 영상은 정상).

**커밋**: `admin-tool` (이 커밋) / `landing` (이 sync 커밋)

---

## 2026-06-03 17:06 — IMC 기획안 v1·v2 git 추적 시작

**무엇을**
- `marketing_division/` IMC 기획안 v1·v2를 git 추적 시작(그간 미추적). v2는 2026-06-03 sync 정합본(§1 인트로 4.4s·상단 헤더 태그라인, §12 2·3번 반영 완료).

**왜**
- 전략 문서가 코드와 함께 버전 관리되도록(백업·이력). ARCHITECTURE/WORKLOG와 동일 정책.

**영향 / 후속**
- 이후 IMC 변경도 `/sync-arch` 대상에 포함. (admin-tool의 `사용방법.txt` 리네임은 사용자 파일이라 미커밋 유지)

**커밋**: `landing` 41953d4

---

## 2026-06-03 17:03 — IMC v2 정합: CTA·서비스명(2번)·HR 화법(3번) + 인트로 최종(4.4s)

**무엇을**
- **인트로 최종화**: 5.0→**4.4s 순차**(셰브론 빠르게 발사·소멸 → "발사 준비 완료" 단독 → 로고+태그라인). 태그라인을 **상단 헤더로 이동**(CASE#·브랜드 아래), 하단 워터마크는 브랜드명.
- **2번 CTA·서비스명 IMC 정합**: `generatePublishKit` 고정댓글/설명에 "현직 HR이 직접 보는 **커리어 런칭 세션(무료 진단)** + 랜딩 링크"(HR멘토그룹) **명시**(전환 직결). 본문(영상)은 소프트 3톤 유지(이원화). 가드레일: 1회 담백·과장/압박/반복 금지(광고처럼 보이면 실패).
- **3번 HR 화법**: SYSTEM_PROMPT 룰 2d 신설 — 시점 HR내부→지원자(주어=상황·현상)·집단 목소리("저희가 수백 건…")·금지패턴("잘 쓰는 법 N가지"·"합격 보장"·HR 화자주어·방법론/툴 이름). QA hard-fail 추가.
- 룰 4곳 동기화(casefile-script.ts+QA / remotion CLAUDE.md / casefile-script.md / casefile-qa.md). v2 기획안 §1·§12 정정(미커밋, marketing_division 미추적).

**왜**
- v2 기획안(2026-06-03)과 코드 정합. 유튜브 유입 전환을 위해 고정댓글에서 무료 진단 명확 안내(과홍보는 가드레일로 통제). 영상 대본은 "현직 HR 집단" 화법으로 차별화.

**영향 / 후속**
- 본문 영상은 광고화 안 됨(소프트 유지). 다음 대본 생성/업로드 키트부터 적용.
- 후속: v2 doc을 git 추적할지 사용자 결정 대기(현재 미추적). BGM(음원 필요)은 별건.

**커밋**: `admin-tool` (이 커밋) / `landing` (이 sync 커밋)

---

## 2026-06-03 11:06 — 브랜드 인트로 V5 발사 심볼 확정 (5.0s 순차 연출)

**무엇을**
- 인트로 로켓을 **V5 추상 발사 심볼**(셰브론 3겹+스파크, SVG)로 교체. 색은 기존(네이비·그린·앰버) 유지.
- 5.0초 **순차 연출**: ① 셰브론 상승 발사(진동 rumble·덜컥 jolt·세로 스피드 스미어) → ② "당신의 커리어, 발사 준비 완료"(크게 강조 후 페이드) → ③ "로켓커리어연구소"(96px·샤인 스윕·글로우, 단독 1초+ 유지) → 아래 "현직 HR과 함께, 커리어 런칭 플랫폼"(글로우+샤인).
- 긴 태그라인은 인트로에서 빼고 **컨텐츠 하단 워터마크**로 상시 노출("현직 HR담당자와 함께, 커리어 런칭 플랫폼").
- `INTRO_SEC` 2.0→5.0, `totalCaseFrames`·씬 시작점에 반영(음성·자막 싱크 유지). 임시 시안 보드(`_RocketSiansBoard`)·Root 등록·테스트 렌더 제거.

**왜**
- 기존 로켓 디자인이 약해 V5로 확정. "로켓 발사=커리어 런칭" 브랜드 컨셉 강화. 사용자 피드백 반영(상승감·살아있는 진동·로고 노출 시간·태그라인 강조).

**영향 / 후속**
- 모든 CaseFileVideo 앞에 5.0s 인트로 추가(영상 길이 +5s). 렌더·TTS·구조 로직 무관.
- 검증: 스틸·mp4 프리뷰 반복 확인 완료. 진동 강도 사용자 요청대로 약화(rumble 9→4.5).
- **미구현(승인됨, 후속)**: 2번 CTA·서비스명 IMC 정합 / 3번 HR 화법 룰 — 다음에 코드+스킬 4곳 동기화 예정.

**커밋**: `admin-tool` 9c0c2d7 (branch feature/marketplace-phase1) / `landing` (이 sync 커밋)

---

## 2026-06-03 10:16 — 벤치마크 변칙 주입 + 브랜드 인트로 + IMC 정렬 결정

**무엇을**
- 트렌드 벤치마크 주입을 **변칙적**으로 재작성(랜덤 표본·랜덤 강조항목·리믹스 지시 회전) → 결과 획일화 차단. [커밋됨]
- 모든 영상 앞 **로켓 발사 브랜드 인트로(1.4s)** 추가(IntroScene, SVG·글리프안전, 기존 팔레트 유지). [커밋됨 — 로켓 모양/태그라인은 확정 대기]
- IMC 기획안(`marketing_division/로켓커리어연구소_IMC_마케팅기획안_v1.md`) 검토 → 영상 카피·컨셉 정렬 방향 도출.

**결정 (사용자 확정)**
- 🎨 **영상 팔레트는 기존 유지**(네이비 #1B2B4B / 그린 #27AE60 / 앰버 #FFC83D). IMC 브랜드북의 크림골드(#E8D5A3)는 **영상에 미사용 확정**(시안 V3·V4 폐기). 로켓은 색 유지 + **모양만** 변경.
- ✅ **CTA·서비스명 IMC 정합** 승인 — 본문은 소프트 유지, 고정댓글/설명은 "커리어 런칭 세션(무료)·HR멘토그룹·랜딩 링크"로. ("런칭"=로켓 발사 컨셉 시너지) → 구현 대기.
- ✅ **훅·나레이션 IMC 화법** 승인 — 시점 HR내부→지원자(주어=상황/현상), 집단 목소리("저희가 수백 건"), 금지패턴("~잘 쓰는 법 N가지"·"합격 보장"·HR 주인공) → 구현 대기.
- 캐치프레이즈(인트로 태그라인) 후보 압축 중: "현직 HR과 함께, 커리어 런칭" / "제대로 쏘아 올리는 커리어 런칭 플랫폼" / "당신의 커리어, 발사 준비 완료".

**영향 / 후속**
- 후속 커밋: 로켓 모양(시안 중 택1) + 태그라인 교체, CTA/화법 룰 4곳 동기화(코드·remotion CLAUDE.md·스킬 md 2개).
- 임시 디자인 도구 `remotion/compositions/_RocketSiansBoard.tsx`(+Root.tsx 등록)는 로켓 확정 후 제거.

**커밋**: `admin-tool` 45e670d (branch feature/marketplace-phase1) / `landing` (이 sync 커밋)

---

## 2026-06-01 20:52 — 내레이션 훅 생성 강화 (케이스 고유 + 다양한 유형)

**무엇을**
- 훅이 케이스마다 비슷하던 문제 해결. `casefile-script.ts` SYSTEM_PROMPT 룰10 전면 개편: 범용 문구("열심히 썼는데 왜 떨어질까요" 류) 기본값 금지, 그 케이스의 고유 디테일(직무·구체 실수·전환 수치·상황) 의무화.
- hook_candidates 3개를 **5가지 유형**(①충격 수치/결과 ②반전 ③구체 실수 지적 ④손실 강조 ⑤직무 특정 공감)으로 서로 다르게 생성.
- 룰3·유저프롬프트에서 클리셰 시드 제거 + "이 케이스의 가장 강력한 한 방으로 후킹" 지시. QA 관점3에 훅 케이스-고유성 점검 추가(약하면 revise가 개선).
- 동기화 4곳: 코드 SYSTEM_PROMPT / `remotion/CLAUDE.md` 룰8 / `.claude/commands/casefile-script.md`·`casefile-qa.md`.

**왜**
- 모든 영상 초기 훅이 거의 같아 후킹력 약함 → 케이스별로 강력하게 차별화.

**영향 / 후속**
- 대본 생성 룰만 변경(구조·렌더·TTS 무관). 다음 ③ 대본 생성부터 적용.
- ⚠️ 이 세션 외 다른 세션에서 멘토 포털·마켓플레이스 쇼케이스 작업 별도 진행됨(이 항목은 **훅 변경만** 기록). admin-tool 커밋은 `feature/marketplace-phase1` 브랜치 위에 올라감.

**커밋**: `admin-tool` 0ea3b39 (branch feature/marketplace-phase1) / `landing` bb4dd77

---

## 2026-05-30 12:32 — 유튜브 업로드 키트 (렌더 후 부가 ⑦)

**무엇을**
- 렌더(⑥) 후 `⑦ 업로드 키트 생성` 버튼 — 확정 대본으로 **제목 3안 / 설명란 / 랜딩 후킹 고정댓글 / 해시태그** 생성.
- `generatePublishKit`(casefile-script.ts) + 신규 라우트 `/api/local-studio/publish-kit`. UI는 편집 + 클립보드 복사. 제작 이력(`_studio-index.json`)에 kit 병합 저장 → 케이스 재선택 시 복원.
- 랜딩 URL: `NEXT_PUBLIC_LANDING_URL`(기본 `rocketcareer-landing.vercel.app`).

**왜**
- 영상 제작 후 유튜브 업로드(제목·설명·고정댓글)까지 한 화면에서 끝내고, 댓글로 랜딩 진단 유입을 후킹하려고. 기존 로직 무수정 부가 작업.

**영향 / 후속**
- ③~⑥ 플로우·렌더·TTS 무수정. local-studio API 7→8종(publish-kit 추가).
- 사용자 검증 완료(정상 작동, 랜딩 URL 확인). 후속: 실제 업로드해 제목·고정댓글 전환율 관찰.

**커밋**: `admin-tool` 8d8f4e8 / `landing` (이 sync 커밋)

---

## 2026-05-30 12:13 — 로컬 스튜디오 나레이션 우선 분리 플로우 + 다수 버그 수정

**무엇을**
- **제작 순서 분리(나레이션 우선)**: ③대본(나레이션) 생성 → 수정 → ④자막 생성(`generateSubtitlesFromNarration`, 나레이션 기준·before/after 원문 유지) → 수정 → ⑤음성(TTS, `generateCaseFileTTS`) → ⑥렌더. 파이프라인 `tts:false` 옵션·`/tts`·`/subtitles` 라우트 신설.
- **QA 권고 실제 반영**: `reviseCaseFileScript`로 검수 issues를 대본에 반영 후 노출(표시만 X).
- **제작 이력에 대본 저장**: `_studio-index.json`에 props·ttsScenes 저장 → 케이스 재선택 시 대본·자막·영상 복원.
- **버그 수정 3종**: ① preview Node스트림→Buffer 반환(미리보기 무한 스피너) ② TTS폴더·파일명 caseId(`caseSlug`) 기준(케이스 간 음성/영상 충돌) ③ JSON 출력 견고화(프리필 미지원 모델 대응 — 강한 JSON지시+1회 재시도).
- **dev 안정화**: webpack 메모리 캐시(`.next` 캐시 손상 방지) + `predev` 포트가드(`scripts/free-port.mjs`, 중복 dev 차단).

**왜**
- 자막·나레이션이 따로 생성돼 두 번 편집하는 번거로움 → 나레이션을 기준으로 자막이 따라오게.
- QA가 권고만 하고 안 고쳐 써먹지 못함 / case_number가 caseId와 어긋나 영상·음성이 다른 케이스로 섞임 / 중복 dev·캐시 손상으로 ENOENT 반복.

**영향 / 후속**
- 정상 `case-NNN` id는 파일명 동일 → 기존 영상 무영향. 충돌났던 stale 인덱스 항목 1건 정리함.
- `/admin/local-studio` API 6→7종(subtitles 추가). 사용자 실제 ③~⑥ 플로우 검증 완료(음성·자막·영상 일치 확인).
- 후속: 새 CTA 톤·분리 플로우로 영상 몇 편 만들며 줄바꿈·품질 체감.

**커밋**: `admin-tool` 66d938b / `landing` (이 sync 커밋)

---

## 2026-05-30 01:16 — 로컬 스튜디오 제작 이력 영속화 + CTA 톤 개편(진단 유도)

**무엇을**
- 로컬 스튜디오 **제작 이력 영속화**: 렌더 시 `out/_studio-index.json`에 caseId별 기록(파일·케이스번호·훅·직무·시각). 신규 `GET /api/local-studio/renders`(mp4 잔존분만). 목록에 `✅ 제작됨` 배지+제작일시, 케이스 전환 후에도 기존 영상 미리보기 자동 복원(대본 생성 전에도 표시).
- **CTA 톤 개편**: "첨삭" 직접 권유 → **커리어 진단/점검·고민** 프레이밍 3톤(점검 권유/호기심 자가진단/공감 셀프체크) 중 케이스에 맞는 1개 변주. 짧게(한 줄 ≤18자·2~3줄). 저장+댓글 2요소·시리즈 예고 금지 유지. QA hard-fail에 "첨삭 직접권유·줄깨짐 길이" 추가.
- 룰 4곳 동기화: `casefile-script.ts`(룰8·JSON예시·QA) / `remotion/CLAUDE.md`(룰6) / `.claude/commands/casefile-script.md`·`casefile-qa.md`.

**왜**
- 다른 케이스 누르면 영상이 사라지고 제작 이력도 안 보여 혼선 → 이력 박제·복원.
- 마지막 후킹을 첨삭 직접 언급보다 커리어 전반 진단 유도가 전환에 유리(사용자 판단). 영상 여러 개 양산이라 톤 변주 + 자막 줄 안깨지게 짧게.

**영향 / 후속**
- `out/_studio-index.json`은 로컬 산출물(.gitignore 대상, 커밋 안 함). mp4 수동 삭제 시 `renders`가 자동 제외.
- CTA 톤은 다음 대본 생성부터 적용 — 실제 렌더로 줄바꿈·길이 체감 후 미세조정 권장.
- 검증: `tsc --noEmit` 통과, dev 서버에서 `/renders` 200·케이스 27 대본 생성 정상.

**커밋**: (미커밋) — `admin-tool` page.tsx·render/route·casefile-script.ts·remotion/CLAUDE.md + 신규 renders/route.ts / `landing` casefile-script.md·casefile-qa.md·WORKLOG.md·ARCHITECTURE.md

---

## 2026-05-30 01:00 — CTA 씬 효과 강화 + 시리즈 예고 제거 + sync-arch 작업로그 기능

**무엇을**
- CTAScene(`remotion/compositions/CaseFileVideo.tsx`) 효과 강화: 헤드 등장 spring + 펄스/글로우 호흡, "저장" 앰버 칩 + boxShadow 글로우 호흡, 카드 테두리 글로우, 댓글 칩 blink + 화살표 bob, 펄스 ring 2개. (밋밋함 해소, 저장·댓글 유도 시선 집중)
- 대본 룰에서 **시리즈 예고("다음편은 ~~편") 금지** 명문화 — `casefile-script.ts`(생성 룰8·QA hard-fail), `remotion/CLAUDE.md`, `.claude/commands/casefile-script.md`·`casefile-qa.md` 4곳 동시 갱신. "저장해두고 다시 보세요" 류로 대체.
- `/sync-arch` 스킬에 **WORKLOG.md 누적 기록 절차** 추가(타임스탬프·커밋 근거·append-only) → WORKLOG.md 신설·운영 시작.
- 운영: dev 서버 `.next` 캐시 손상(`ENOENT _document.js`, 프로덕션 빌드+dev 산출물 혼재) 복구 — 포트3000 종료·`.next` 삭제·재기동.

**왜**
- 회차가 유동적이라 "다음편" 약속을 지킬 수 없어 신뢰 리스크 → 룰로 영구 차단.
- CTA가 영상 마지막 전환 포인트인데 정적이라 저장·댓글 행동 유도가 약함.
- 문서 "현재 상태"만으로는 *어떤 과정을 거쳤는지* 추적 불가 → 시간순 히스토리 필요.

**영향 / 후속**
- CaseFileVideo.tsx 변경 — 로컬 렌더는 즉석 번들이라 즉시 반영. (Lambda 경로 쓸 일 있으면 재배포 필요하나 현재 로컬 전용이라 무관.)
- 룰 변경 3곳 동기화 규칙(코드 프롬프트/remotion CLAUDE.md/스킬 md) 준수함.
- 검증: case-002 실제 로컬 렌더로 QA 통과·CTA 나레이션 시리즈 예고 없음·효과 프레임 확인 완료.

**커밋**: `admin-tool` fa2986f / `landing` 2b98b68  (※ `.next` 복구는 운영 작업, 미커밋)

---

## 2026-05-30 00:34 — CaseFile 영상 로컬 렌더 전환 + 로컬 스튜디오 웹 패널 신설

**무엇을**
- 영상 렌더를 AWS Lambda → **완전 로컬**(`@remotion/bundler`+`@remotion/renderer`)로 전환. CLI `scripts/make-case.mts`(`npm run make:case`) 신설.
- QA 게이트: 검수 최종 실패 시 TTS 전에 `QAGateError` throw (`runCaseFilePipeline({ failOnQA })`).
- 폰트 최적화(`remotion/lib/fonts.ts`): weight(400/600/700/800/900)+subset(korean/latin) 제한 → 네트워크 요청 1116→605.
- 어드민 검수 UI(`review/page.tsx`)에서 영상 생성 버튼·폴링·롱폼 컬럼·음성 노브 제거 (검수/승인/스킵/편집만).
- **로컬 스튜디오 웹 패널** `/admin/local-studio` + API 4종(`app/api/local-studio/{cases,script,render,preview}`) 신설 — 케이스선택→설정→대본생성→편집(훅후보/before/after/diff)→로컬렌더→미리보기. 전부 `NODE_ENV=production`이면 403(dev 전용).

**왜**
- 배포된 Lambda serveUrl이 구형 번들(ShortsVideo만)이라 `Could not find composition CaseFileVideo` 발생. 로컬은 매번 즉석 번들 → 항상 최신 컴포지션. Lambda 비용·serveUrl 재배포 부담도 제거.
- 무거운 영상 작업은 로컬, 일상 운영(검수·큐레이션)은 온라인 어드민으로 환경 분리.

**영향 / 후속**
- 온라인 어드민에는 영상 기능 없음(403). 영상은 로컬에서만. `make-case.mts`와 웹 패널은 동일 파이프라인(`runCaseFilePipeline`) 재사용(SoT).
- Vercel hobby 플랜 maxDuration 상한(300)에 맞춰 render 라우트 600→300 수정(빌드 통과용, 프로덕션 미실행).
- 미검증: QA 통과 케이스의 happy-path 로컬 렌더(Claude+TTS 비용 때문에 수동 확인 대기).
- 확장 여지: 향후 온라인 영상 생성은 `enqueueCaseFileJob()` seam으로 큐 전환 가능(준비됨).

**커밋**: `admin-tool` 71d0d4b · f678d60 · 16c9a58 / `landing` 81680b9 · 5bb54a5

---
