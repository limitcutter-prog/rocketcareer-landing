# 로켓커리어연구소 — 마스터 아키텍처 인덱스

> **모든 작업 시작 전 이 파일 읽기 필수. 완료 후 반드시 업데이트.**
> 상세 구현은 각 모듈의 `CLAUDE.md` 참조.
> `admin-tool/` 기준 작업 시 ARCHITECTURE.md 경로: `../ARCHITECTURE.md`

마지막 업데이트: 2026-06-21 (**시장 리포트 어드민 + 멘토 온보딩 + 메뉴 정비 (/order 8건 배치)** — ① #1·#2 시장조사 메일을 어드민 누적·열람 + 수동 발송: `lib/market-research`(크론·수동 공유·🧭부서별 시사점) · `/admin/market-research` · `/api/market-research/{run,list}` · `market_reports`(`supabase-market-reports.sql`)·`0084f37`. ② #7 네비 '재무'→'매출·재무'+'시장 리포트'. ③ #4 멘토 온보딩 메일 `lib/mentor-onboarding`·`/api/mentors/[id]/onboard`+멘토 상세 버튼(정산/세무 #6 포함)·`60344c0`. ④ #3(중간)·#5·#8 LEDGER L-0011~L-0016 순차화. ⚠️ **배포 블로커**: Vercel `CRON_SECRET` 한글값 → HTTP헤더 ASCII 위반 빌드실패(사장 ASCII 교체 후 반영, 프로덕션은 `6b51d74` 정상). 🔒 FROZEN 미접촉) ←→ 2026-06-21 (**S1→S2 회차 버그픽스 + 판매분석 툴 + 아침 시장조사 Cron 메일 (프로덕션 배포)** — ① 멘티 포털 S1 완료 시 S2 미활성 버그: 회차 PATCH가 `done` 전환 시 다음 회차(seq+1) `locked→assignment_pending` 연쇄 + 멘티 화면 잠금 순차전용 보정(레거시 즉시 해제)·`dd1e543`. ② 판매분석 `/admin/finance` 확장 — `SalesAnalysis`(현황분석 버튼→`/api/finance/commentary` 영업·사업개발 관점 AI 코멘트, 상품×월 매트릭스·필터) + `lib/finance-data` 집계헬퍼(`financeSummary`·`productMonthMatrix`)·`FINANCE_SALES`(상품×월, 원본 `.xls` 입수 시)·`/api/finance` OWNER_ONLY·`aa10e3b`. ③ 아침 시장조사 자동메일 `vercel.json` crons(08:00 KST)→`/api/cron/market-research`(CRON_SECRET 검증, 미들웨어 `/api/cron/` 공개)→Claude `web_search` 3C/4P + 내부 판매현황 → `sendMail(OWNER_COPY)`·`6b51d74`. 🔒 FROZEN `lib/anthropic` 미접촉(신규 클라이언트). tsc 0·next build 0. 사장 선행: Vercel `CRON_SECRET` 추가·크몽 매출 `.xls` 재첨부(상품×월)) ←→ 2026-06-21 (**상품 카탈로그 + 재무 대시보드 + 멘토 노출/사진 편집 + 메일 정비 (프로덕션 배포)** — ① 상품 SoT `products` 테이블 + `/admin/products` CRUD(타이틀·소개·가격·세션·노출)·`a5fd1c7`(SQL `supabase-products.sql`). ② 재무 `/admin/finance`(**사장 전용 OWNER_ONLY**) — 크몽 수수료·광고 세금계산서 1~5월 집계 + AI 운용비 → 월별 손익·추이·상품별·세무, `lib/finance-data.ts` SoT(임베드)·`bc7da08`. ③ 멘토 상세 `is_visible`·photo·연차 사장 편집(`lib/mentors` 매핑). ④ 멘토 화면 `app/admin/mentees/[id]/SubmissionView.tsx`(멘티 제출 항목 구조화)·회차 첨부 열람·`6275ef5`. ⑤ 메일 링크 `APP_URL` 프로덕션 고정 + 멘티/멘토 메일 `OWNER_COPY` BCC — 🔒 FROZEN 진단메일(`submit-diagnosis`·`inbox/send-email`·`send-email`)은 **사장 승인** 하 링크/BCC만 최소수정(`30bc873`, CLAUDE.md 예외기록). RBAC 멘토차단 실측(어드민 307·API 401). org LEDGER L-0007~L-0010 /standup 처리) ←→ 2026-06-19 (**멘티 포털 후속 — 리포트 사전정보 반영·회차 명칭 통일·멘토 열람/편집·S1/S2 양식 보강 (프로덕션 배포·검증)** — ① 리포트 AI가 멘티 제출(`assignment_input`) + 멘토 소통(재량 판단·과제 안내)을 근거로 분석(S0·S3·S4 포함, STT 기존)·`6f4ed95`. ② 회차 명칭 SoT `lib/mentoring/session-labels.ts`(배지 OT/Session 1~4 · 명칭 진단/타깃 그룹화/경험 구체화/서류 구체화/보강 및 보완) 멘토·멘티·관리자·리포트·메일 공통 + JourneyPanel '멘티 제출 자료'(텍스트+첨부 서명URL)·`60b7fa9`. ③ 멘토 AI 리포트 직접 편집 `app/admin/mentees/[id]/ReportEditor.tsx`(PATCH `final`) + 회차별 첨부 열람 `app/api/mentees/[id]/journey/sessions/[sid]/files` + S1/S2 폼 보강(점수 제거·양식 2종 반영)·`64bfb63`. 전부 main FF 배포·prod 검증(매직링크→제출→직렬화). SQL 추가 없음) ←→ 2026-06-18 (**멘티 포털(멘티 화면) 신규 구축 — S0~S4 풀 딜리버리, Phase A~C 코드완료·tsc 0** — 멘토 화면(JourneyPanel)의 멘티 카운터파트 신설. 멘티가 **이메일 매직링크(passwordless·`mentee_auth`, mentees 스키마 무수정·stateless)**로 로그인해 **회차별 자료를 구조화 제출**(`mentee_submission` JSONB → `assignment_input` 직렬화 → 멘토 JourneyPanel·AI 그룹화/합리화 제안 **무수정 동작**)하고 **리포트를 열람**(sent만). 신규 `app/mentee/*`(login·dashboard·forms)·`app/api/mentee/*`(login/verify/logout/me·sessions/[sid] 제출·files·report+pdf)·`lib/mentee-portal.ts`(인증·소유권·노출 화이트리스트·직렬화)·`lib/auth.ts` 멘티 헬퍼·`middleware.ts` 멘티 게이트. 멘토측 글루: `JourneyPanel.tsx` "멘티 초대"(매직링크) + S1/S2 "멘티 제출됨" 배지 + `journey/invite`. 입력 명세 = SOP §3·운영스크립트 회차. 🔒 FROZEN(mentees/diagnoses/contracts) 읽기조인만 + 전 라우트 소유권 검증(token menteeId===journey.mentee_id) + `safeSession` 노출통제(judgment_note·ai_suggestion·stt_text 비노출). SQL `supabase-mentee-portal.sql`(case_sessions.mentee_submission/submitted_at·files.session_id, **미적용**). 후속(사장 승인): SQL 적용·라이브 스모크·메일발송) ←→ 2026-06-16 (**마켓플레이스 생애주기 재설계 P1·P2 (프로덕션 배포·검증)** — 멘토–멘티 데이터 파편화(멘토 `mentors`↔`mentor_profiles`, 멘티 `mentees`↔`mentee_applications`, 인증 3분할) 통합 시작. **P1 멘토 단일화(#1·#2)**: `/api/mentors`(GET·POST·[id])를 `mentor_profiles` 단일 풀로 전환 + `lib/mentors.ts` 정규화(expertise→specialties·headline→소속·approved→active). 멘토 심사 승인분이 멘토 관리·진단 멘토지정 드롭다운에 즉시 노출. 멘티 배정 0건이라 mentor_id 재매핑 불필요. 프로덕션 검증: /api/mentors 1→7명 전환. **P2 로그인·권한 통합(#3)**: 통합 `/login`(사장 PIN·스태프/멘토 email+pw 단일 진입·역할 라우팅), `/staff-login`·`/mentor/login`→`/login` 리다이렉트, `/api/auth/me` 멘토 역할 추가+공개. 사장 PIN·미들웨어 owner 분기 무변경→잠금0(프로덕션 사장 로그인 200 확인). 로컬 E2E: 사장 me=owner·스태프 허용/사장전용307·멘토 me=mentor·포털200·/admin307. **후속**: P3(멘티 진단 단일 퍼널·case 자동생성)·P4(소통·파일) 미착수. ⚠️ 로컬 dev `.next` Windows 손상 반복 → 검증은 실데이터·프로덕션 우회. 커밋: admin-tool `fdfcad3`(P1)·`9ebd99e`(P2)) ←→ 2026-06-14 (**RBAC 권한 3층 + 보안 강화 + 멘토 가입 동의 + 진단/리포트 개선 (프로덕션 배포·검증)** — ① **#6 RBAC**: 사장(PIN, 무변경)/운영스태프(`staff_users`·`/staff-login`·`staff_auth`)/멘토 3층. `middleware`가 사장전용(조직운영·멘토관리·멘토심사·스태프관리)만 스태프 차단(denylist `lib/permissions.ts`), 네비 역할필터. `/admin/staff`(사장)·`/api/staff`. E2E: 사장 전체200·스태프 허용200/사장전용307·API403. 프로덕션 사장 로그인 무손상 확인. ② **보안**: 전 public 테이블 RLS on(service_role 우회·무중단)+`mentee-files` 버킷 private+STT 녹취 경로저장(public URL 제거). ③ **멘토 가입**: `/showcase/apply`에 PII 수집·이용 동의 필수(`mentor_applications.consent_*`)—동의 미전송으로 막혔던 신청 정상화. 초대링크 `/showcase/apply` 통일. ④ **진단 리포트**: AI 추천상품 STEP4 자동체크·선택 반영 재생성, 추천서비스 박스 제거(본문 솔루션 중첩·🔒메일 사장승인 수정), 카카오 문의 버튼(/report 페이지). ⑤ **멘토링 리포트 메일** 카드 레이아웃 재설계+카카오(이메일·PDF). ⑥ **조직 콘솔** 협업 R/R 로그 표시·즉시반영 수정. ⑦ **랜딩** 멘토 매칭 보조 링크. 커밋: admin-tool `54de08e`·`b243e0e`·`1f397a3` / landing `488cab1`) ←→ 2026-06-14 (**모듈13 STT 파이프라인 구현** — Google STT + ffmpeg 5분 청크 분할, 2트랙(A=텍스트 직접입력·B=파일 업로드 자동변환). `lib/mentoring/stt.ts`(JWT 인증·청크 순차 STT 신규)·`stt/route.ts`(PATCH Track A · POST Track B · maxDuration=300 신규)·`report-generator.ts`(sttText 6번째 인자·프롬프트 상단 주입·max_tokens 5000)·`report/route.ts`(stt_text 조회·전달)·`JourneyPanel.tsx`(STT 섹션 UI · Session 인터페이스 recording_url/stt_text 추가). tsc 에러 0. 커밋: 미커밋) ←→ 2026-06-13 (**모듈13·14 프로덕션 배포** — admin-tool `5d3e1d0` → main FF 푸시로 Vercel Production 빌드 ● Ready(3m). 배포: 모듈13(멘토링 딜리버리·리포트, 내부 어드민) + 모듈14 콘솔(prod 숨김 — 네비 `devOnly` 필터·페이지 dev 전용 안내·API 403) + casefile 수정(`a33ec1c`). 전부 admin 인증 뒤·고객 노출 0·🔒 FROZEN 무수정. 콘솔 풀스택 검증(로그인 200·overview 8부서·agents 6섹션·page 200) 통과) ←→ 2026-06-13 (**운영 콘솔(회사 운영 콕핏) 구축** — 모듈14 org/ 파일을 GUI로 보고 편집하는 dev 전용 대시보드. `admin-tool/lib/org/`(파서 7) + `app/api/org/`(라우트 7, prod 403 가드) + `app/admin/org/`(조직 맵·오더 보드·목표 + 부서 드로어). 파일=단일 진실원천(Node fs로 org/·.claude/agents 직접 r/w), 분리형(설정=콘솔/실행=/standup). LEDGER append-only·고위험 자동 승인대기. `scripts/verify-org.mts` 실파일 라운드트립 검증(에이전트 무손실·append-only 보존·고위험 게이트), tsc 0) ←→ 2026-06-13 (**리포트 버그픽스·표현 개선** — `repairTruncatedJson`+max_tokens 6000으로 S2 JSON 잘림 502 해소. `parseJson`에 가운뎃점(·)→쉼표+공백 후처리. 로드맵 레이블 변경: S2=경험 구체화/S3=심화과정/S4=심화과정(추가). `report-pdf.tsx`·`report-generator.ts` 동기화) ←→ 2026-06-13 (**AI 운영체계(모듈14) 구축 + SQL 마이그레이션(L-0001) 완료 + 퍼널 측정 체계 + 랜딩페이지 수정** — org/ 파일 7종·에이전트 8개·커맨드 4개 신설. case_journeys·case_sessions·lead_analytics·bitly_clicks·trend_benchmarks·content_priority DB 실행 완료. lead_analytics 트리거 LIVE(EXCEPTION WHEN OTHERS 하드닝, diagnoses FROZEN 무손상). 랜딩페이지 v2: no-cors 제거·concern 필드 수정·URL 파라미터 추적 추가(bit.ly/rcl-shorts → ?ref=youtube_shorts → source_channel 자동 매핑). Bitly 토큰 등록 완료. LEDGER L-0001 검증) ←→ 2026-06-13 (**S1/S2 리포트 구조 재설계** — `S1GroupCard`·`S2ExperienceMapping`·`RoadmapInfo` 신규 인터페이스. S1=그룹별 접근전략 카드, S2=경험×그룹 포지셔닝 매트릭스, 전 회차=5단계 로드맵 인디케이터. S2 리포트 생성 시 S1 ai_suggestion 자동참조. `RESEND_FROM_EMAIL=noreply@rocketcareer.co.kr` 반영. tsc 에러 0) ←→ 2026-06-11 (**모듈 13 멘토링 운영(딜리버리) 시스템 등재** — 두 운영 문서(SOP v1.1·운영스크립트 v1.0, `admin-tool/mentoring tool/files/`) 평가 → 어드민의 "네 번째 축"인 멘토링 실행(딜리버리)이 비어있음 확인(녹취→STT→리포트 **코드 0건**, `lib/pdf.ts` generatePdf **throw 비활성**). 모듈 13 신규 등재(📋 PLANNED). **상품모델 결정**: `makeup`="회사선택→이력서 Make-up"(200,000원)=SOP 5세션(S0 OT~S4). **FROZEN 우회**: 회차·리포트는 신규 테이블(case_journeys/case_sessions/session_reports/case_artifacts) 분리, 모듈1(mentees/contracts/session_notes) 무수정·FK 읽기만. 계약 문서: 루트 `PLAN.md`) ←→ 2026-06-07 (**모듈 12 마켓플레이스 프로덕션 배포** — admin-tool `feature/marketplace-phase1` → main FF 머지·푸시(`0abd512`)로 Vercel 자동배포 + 신규 SQL 4종(setup→apply→auth→seed) Supabase 실행 완료(11테이블·멘토 로그인 컬럼·시드). 모듈 12 🚧 BUILT·미배포 → ✅ ACTIVE. 미결 'SQL 미실행'·'머지/배포 결정' 해소. IMC v2 §1 'Track B 미운영' → 'Phase 1 라이브'로 정합. ⚠️ main 직접 푸시가 auto모드 가드레일에 막혀 사용자가 `Bash(git push:*)` 권한 부여 후 단독 푸시로 진행) ←→ 2026-06-05 (**모듈 12 멘토링 마켓플레이스 등재 — 거버넌스 표류 시정** — `feature/marketplace-phase1`에만 있어 그간 미등재였던 양면 마켓플레이스(공개 쇼케이스+멘토 온보딩+멘토 포털+거래, 신규 테이블 11개)를 모듈 12로 인벤토리·상세·미결SQL에 등재. ⚠️ IMC v2 'Track B 미운영'·모듈 10 '멘토포털 미시작'과 상태 모순 표기. 미배포(main 미머지)·SQL 4종 미실행. 미머지 브랜치를 `/sync-arch`가 못 보던 맹점 노출) ←→ 2026-06-03 (**마케팅 전략 레이어(IMC) 거버넌스 등재** — `marketing_division/로켓커리어연구소_IMC_마케팅기획안_v2.md`를 전략 SoT로 "📣 마케팅 전략 레이어" 섹션 신설(전략결정→코드 매핑·정합 규칙), `/sync-arch`에 마케팅 정합 절차·출력 라인 추가) ←→ 2026-05-31 (모듈 8 인코딩 파이프라인 로직 박제: 표 추출 추가·max_tokens 16000·실패유형 매핑·왜인코딩하는지 3개 다운스트림 명문화. cases_career 29건/rules 173건/임베딩 29건 갱신) ←→ 2026-05-30 (CaseFile 영상 **완전 로컬 렌더 전환** — `scripts/make-case.mts`+`npm run make:case` / QA 게이트(`failOnQA`) / 폰트 weight 제한 최적화 / 어드민 웹 영상 UI 제거(쇼츠 버튼·롱폼 컬럼·케이스파일 웹 버튼) / **로컬 웹 패널 `/admin/local-studio` 신설**(dev 전용 — 케이스선택→대본생성→편집→로컬렌더→미리보기). **로컬 스튜디오 제작 이력 영속화**(renders API+배지+미리보기 복원) / **CTA 톤 개편**(첨삭 직접언급→커리어 진단/점검 3톤 변주·짧게) / **로컬 스튜디오 나레이션 우선 분리 플로우**(나레이션→자막→음성→렌더, QA 권고 대본 반영, caseId 기준 충돌 차단, 미리보기 버그·dev 안정화 수정) / **유튜브 업로드 키트**(렌더 후 ⑦ — 제목3·설명·랜딩 후킹 고정댓글))

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
| 10 | Phase C (Auth/포털) | 🔶 표면 구현·Auth전환 대기 | — | `admin-tool/app/mentor/`(모듈12)·`admin-tool/app/mentee/`(모듈13) |
| 11 | 리포트/PDF/Sheets | ✅ ACTIVE | admin-upload-agent | `admin-tool/app/api/generate-pdf/`, `app/api/sheets/`, `lib/pdf.ts` |
| 12 | 멘토링 마켓플레이스 (양면) | ✅ ACTIVE (배포 2026-06-07) | — (미지정) | `admin-tool/app/showcase/`, `app/mentor/`, `app/api/{showcase,mentor}/`, `supabase-marketplace-*.sql` |
| 13 | 멘토링 운영 (딜리버리) | ✅ 2단계 배포 (2026-06-13, `5d3e1d0`) — 리포트 react-pdf | — (미지정) | `admin-tool/app/admin/mentees/[id]/JourneyPanel.tsx`, `app/api/mentees/[id]/journey/`, `lib/mentoring/`, `supabase-mentoring-ops-setup.sql` |
| 14 | AI 운영체계 (자율 협업) + 운영 콘솔 | ✅ ACTIVE (2026-06-13) | chief-of-staff·career-lab-lead·6본부장 | `org/`(PROTOCOL·IMPACT_MAP·CHARTER·OBJECTIVES·LEDGER·ORG_CHART·state), `.claude/agents/*-head.md`, `.claude/commands/{order,standup,strategy-review,org-audit}.md`, **콘솔(dev)** `admin-tool/app/admin/org/`·`app/api/org/`·`lib/org/` |

---

## 📣 마케팅 전략 레이어 (IMC) — 위 코드 모듈들의 상위 의도

> 모듈 2·5·7·9의 코드는 **마케팅 IMC 전략의 구현체**다. 전략 자체의 단일 진실 소스(SoT):
> **`marketing_division/로켓커리어연구소_IMC_마케팅기획안_v2.md`**
> (브랜드 아이덴티티·메시지·채널·콘텐츠·퍼널·KPI·SOP). v1은 원본 보존.
> 스코프: 커리어 서비스(Track A)만. 전월세 안심 리포트는 별개 도메인(제외).

**전략 결정 → 구현 매핑 (변경 시 양쪽 동기화):**

| IMC 전략 결정 | 구현 위치 (코드·자산) | 상태 / 정합 키 |
|---|---|---|
| BI 컬러 (네이비 `#1B2B4B`·그린 `#27AE60`·앰버 `#FFC83D`) | `style.css`, `remotion/lib/fonts.ts`, `CaseFileVideo.tsx` | 골드크림 `#E8D5A3` 폐기 — 문서·코드 일치 |
| 브랜드 카드 V5 (발사 메타포, **클로징 아웃트로**) | `remotion/compositions/CaseFileVideo.tsx` IntroScene | `INTRO_SEC=4.4` · CTA 뒤 배치(오프닝→클로징) · 상단 헤더 태그라인 |
| 캐치·발사/런칭 메타포 | 인트로 태그라인 + `generatePublishKit` | "당신의 커리어, 발사 준비 완료" |
| 콘텐츠 화법 룰 (HR내부→지원자·집단목소리·금지패턴) | `lib/marketing/casefile-script.ts` SYSTEM_PROMPT 룰2d | ✅ 구현(2026-06-03) · **4곳 동기화** |
| 훅 5유형 (케이스 고유) | `casefile-script.ts` 룰10 | 범용 클리셰 금지 |
| CTA — 본문 진단 3톤 / 고정댓글 명시 (이원화) | `casefile-script.ts` 룰8 + `generatePublishKit` | ✅ 구현(2026-06-03) · 본문 소프트·댓글 명시 |
| 콘텐츠 3대 카테고리·믹스 (포괄2+특화1) | `lib/marketing/content-scorer.ts`, `script-generator.ts` | `content_angle` 기반 |
| 퍼널 (진단→리포트 48h→전환) | 🔒 구간3 + 모듈 9 랜딩 + 모듈 11 리포트 | FROZEN 경로 |
| 페르소나 3개 | (전략 문서 — 콘텐츠 타겟팅 지침) | 코드 직접 매핑 없음 |

> ⚠️ **정합 규칙:** 위 결정을 바꾸면 → ① IMC 기획안 v2 갱신 + ② 해당 코드/자산 수정 + ③ (화법·훅·CTA는) 룰 **4곳 동시 갱신**(`casefile-script.ts` · `casefile-qa.md` · `remotion/CLAUDE.md` · `casefile-script.md`) + ④ `/sync-arch`. 코드만 바뀌고 전략 문서가 안 따라오면(또는 반대) `/sync-arch`가 "📣 마케팅 정합 표류"로 리포트한다.

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

lead_analytics   — 유입 분석 (✅ 트리거 LIVE 2026-06-13 — EXCEPTION WHEN OTHERS 하드닝)
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
🚀 브랜드 카드(2026-06-03, 영상 **맨 뒤 클로징 아웃트로** 4.4s, 순차): 발사 심볼(셰브론 V5, SVG·글리프안전)
  ① 빠른 상승 발사(진동·덜컥·스피드 스미어) → ② "당신의 커리어, 발사 준비 완료" → ③ "로켓커리어연구소"(샤인/글로우) + "현직 HR과 함께, 커리어 런칭 플랫폼"으로 마무리.
  ⚠️ **오프닝→클로징 이동(2026-06-03)**: 오프닝에 두니 쇼츠 초반 집중도가 깎여 **영상 끝**으로 이동. 시작은 기존대로 HOOK부터.
  INTRO_SEC=4.4(브랜드 카드 길이), totalCaseFrames 반영(음성·자막 싱크 유지). 기존 팔레트 유지.
  **컨텐츠 상단 헤더** = CASE# + 로켓커리어연구소 + "현직 HR담당자와 함께, 커리어 런칭 플랫폼"(상시 노출). 하단 워터마크 = 브랜드명.
구조(5씬+아웃트로): HOOK → CASE CONTEXT → PAIR×N(BEFORE/AFTER) → CLOSING → CTA → **브랜드 카드(클로징)**
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

#### (3-B) 배경음악 BGM (2026-06-03)
| 항목 | 값 | 출처(SoT) |
|---|---|---|
| 설정 | `BGM_CONFIG` (enabled·defaultSrc·volume·fade) | `remotion/lib/bgm.ts` |
| 볼륨 | 인트로 `0.11` → 본문 `0.05` → CTA `0.045`(억제) + 시작/끝 페이드 | `CaseFileVideo.tsx` bgmVolume |
| ⚠️ 원칙 | **BGM은 TTS보다 절대 앞으로 X** (본문 0.05로 TTS 1.0 아래 고정) | — |
| 음원 | Pixabay "Medical Doctor Clinic Background"(prettyjohn1, Corporate) · `public/bgm/`(**git 미추적, 로컬만**) | — |
| 토글 | props `bgm:false` / `BGM_CONFIG.enabled=false`로 즉시 제거(저작권 제한 시) | — |
| ⚠️ 렌더 | staticFile 해석 위해 `bundle({publicDir: public})` 필수(render 라우트·make-case) | — |
| 운영 | 일부공개 업로드 후 저작권 제한 확인 → 뜨면 즉시 제거 | — |

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
| **`/admin/local-studio`** (웹 패널) | 로컬 dev 서버(`app/api/local-studio/*`, API 8종) | **CaseFileVideo v7** | ✅ **권장 경로 (웹)**. dev 전용(NODE_ENV=production이면 403). **나레이션 우선 분리 플로우**: ③`script`(나레이션 생성+QA권고 반영)→④`subtitles`(나레이션 기준 자막, before/after 원문 유지)→⑤`tts`(음성+duration)→⑥`render`(즉석 번들)→`preview`→⑦`publish-kit`(유튜브 제목3·설명·랜딩 후킹 고정댓글). **제작 이력 영속화**(`out/_studio-index.json`+`renders` API): `✅제작됨` 배지·일시 + 케이스 전환 후 대본·자막·영상·키트 복원. **TTS 폴더·파일명 caseId(`caseSlug`) 기준**(충돌 차단). preview Buffer 반환 |
| `POST /casevideo` (Lambda) | (어드민 버튼 제거됨) | CaseFileVideo | ⚠️ 잔존 — 배포된 serveUrl이 **구형(ShortsVideo만)**이라 `Could not find composition CaseFileVideo` 발생. 쓰려면 serveUrl 재배포 필요 |
| `POST /video` (ShortsVideo) | (어드민 버튼 제거됨) | ShortsVideo(구형) | ⚠️ 구형 — UI 제거됨. 백엔드 라우트만 잔존 |

**🔒 정합 결정 (고정):**
1. **CaseFile 영상은 로컬에서만 생성** — CLI `npm run make:case` 또는 웹 패널 `/admin/local-studio`(dev). 둘 다 기존 TS 파이프라인(`runCaseFilePipeline`)을 그대로 재사용(SoT) → 대본·검수·TTS·조립 로직 중복 없음. 웹 패널은 **나레이션 우선 분리 플로우**: 나레이션(ttsScenes) 먼저 생성·편집 → 그에 맞춰 자막(props) 생성·편집 → TTS → 렌더(나레이션만 수정 시 ④ 재생성, 자막만 수정 시 바로 렌더). QA 권고는 `reviseCaseFileScript`로 대본에 반영 후 노출. TTS 폴더·출력 파일명은 caseId(`caseSlug`) 기준으로 충돌 차단.
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

#### 인코딩 파이프라인 로직 (2026-05-26 정리 — 이 절 절대 잊지 말 것)

**왜 인코딩하는가 (3개 다운스트림 동시 공급):**
1. **마케팅 자동화** — `hook_sentence`·`content_angle`·`transformation_score`를 `content-scorer.ts`·`script-generator.ts`가 직접 참조 → 쇼츠/롱폼 양산
2. **RAG 검색** — `embedding` 컬럼으로 `search_similar_cases()` RPC가 유사 케이스 top-N 반환. `ver1-resume-writer` SKILL.md §0가 신규 멘티 경력기술서 작성 전 자동 조회 → 영석님 판단 패턴 주입
3. **판단 규칙 누적** — `rules_career`에 LAYER1~7 패턴/판단/수정안을 `frequency`로 카운트 축적 (HR 휴리스틱의 DB화)

**파이프라인 단계 (`cases/career/case-XXX/` → DB):**
```
① classifier.py
   - Before: 파일명 버전마커 없음
   - After:  V숫자 / 수정 / 완성 / 최종 / henry / FINAL 포함 (최고버전=최종 After)
   - 판별 불가 → _ambiguous/ 이동

② analyze_case.extract_text  ← 콘텐츠 품질의 첫 번째 관문
   - docx: 문단 + 표(table) + 텍스트박스를 문서 순서대로 (★2026-05-26 표 추출 추가)
           이전엔 paragraphs만 읽어 표 기반 완성본이 0자로 누락됐음
   - pdf:  pdfplumber, 이미지PDF는 "이미지 기반 PDF" 에러 반환 (Claude 호출 안 함)
   - 가드: before/after 텍스트가 비면 Claude 호출 전 에러 반환 (API 낭비·DB 오염 방지)

③ analyze_case Claude 호출 (Sonnet 4-6)
   - max_tokens 16000 (★2026-05-26 6000→16000, 풍부한 케이스 truncation 방지)
   - 응답 파싱: ```json 펜스 제거 → json.loads → 실패 시 {...} 정규식 폴백
   - 최종 실패 → output/{type}/analyses/{cid}_raw_fail.txt 디버그 저장 후 에러

④ store_case.py
   - cases_career upsert (id=PK)
   - rules_career frequency 증가 (같은 layer/pattern은 케이스 ID만 추가)
   - sync_cases.py 삭제 시 frequency 감소 → 0이면 is_active=FALSE

⑤ embed_cases (batch_process 내장 자동 호출)
   - paraphrase-multilingual-MiniLM-L12-v2 (384dim, 한국어)
   - cases_career.embedding에 즉시 저장 (코사인 유사도 검색 가능 상태)
```

**알려진 실패 유형 매핑:**
| 증상 | 진단 | 처치 |
|------|------|------|
| docx인데 추출 0자 | 표 기반 완성본인데 표 추출 누락 | ✅ 해결됨 (2026-05-26) |
| raw_fail.txt 끝이 `}` 없이 끊김 | 응답 토큰 한도 초과 | ✅ max_tokens 16000 (2026-05-26) |
| raw_fail.txt 정상 종료인데 json.loads 실패 | 응답 문자열 내부 unescaped 특수문자 | 재시도하면 대개 해결 |
| extract_text가 "이미지 기반 PDF" 에러 | 스캔 PDF, OCR 미적용 | 텍스트 docx 변환 필요 |
| Before 폴더에 영석님 작업본만 존재 | 멘티 원본 부재 | 원본 추가 or 영구 스킵 |
| API 응답 "credit balance too low" | 크레딧 소진 | 충전 후 `--skip-done` 재실행 |

**현재 보류 5건 (2026-05-26):** case-024(특수문자 재시도 대기), case-048(크레딧 충전 대기), case-056·102(이미지 PDF — 원본 docx 필요), case-084(멘티 원본 부재)

---

### ✅ 모듈 9 — 랜딩페이지

**역할:** 유튜브 유입 → 서비스 소개 → 진단 신청 CTA → `/api/submit-diagnosis` 연결

**배포:** `rocketcareer-landing.vercel.app`
**유입 단축 URL:** `https://bit.ly/3RSgWMy` → vercel 배포 URL로 포워딩

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

### 🔶 모듈 10 — Phase C (Auth/포털) — 표면 구현·Supabase Auth 전환 미착수

**역할:** 단일 PIN 로그인 → 역할 기반 접근 제어 (Admin / 멘토 / 멘티 3계층)

> 🆕 **2026-06-18 갱신:** `app/mentor/*`(모듈12)·`app/mentee/*`(모듈13)·통합 `/login`은 **이미 구현됨** — 단 **HMAC 서명 토큰** 기반(멘토 `password_hash` / 멘티 매직링크). 따라서 이 모듈(Phase C)의 잔여 범위는 **Supabase Auth(JWT) 통합 전환 + `user_id` 연결**뿐. 표면은 존재, 인증 백엔드 전환만 미착수.

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

### 🚧 모듈 12 — 멘토링 마켓플레이스 (양면 거래) — BUILT·미배포

> **상태:** ✅ **배포됨 (2026-06-07).** `feature/marketplace-phase1` → origin/main FF 머지·푸시(`0abd512`) → Vercel 자동배포. 신규 SQL 4종(setup→apply→auth→seed) Supabase 실행 완료. PRD "로켓커리어 멘토링 마켓플레이스 Phase 1~3"(PRD 문서 자체는 git 미추적). 단 멘토 인증은 자체 `password_hash`(모듈10 Supabase Auth와 별개) — 통합 방향 미정.
>
> ⚠️ **거버넌스 모순 (정합 필요):**
> - ~~**IMC v2 §1** "Track B 미운영"~~ → ✅ **정합 완료 (2026-06-07)**: 배포로 Track B Phase 1 라이브 → IMC v2 §1을 "Phase 1 라이브"로 갱신함.
> - **모듈 10(Phase C)**은 멘토 포털 "📋 미시작"이라 명시 — 그러나 `/mentor` 포털·멘토 로그인이 이미 구현됨. 단 **Supabase Auth(JWT)가 아니라 자체 `password_hash` 로그인**이라 모듈 10 설계와 별개 인증 → 통합 방향 결정 필요.

**역할:** 공개 멘토 쇼케이스 + 멘토 온보딩(지원→심사→승인) + 멘토 포털(받은 신청 수락/거절·프로필) + 양면 거래(신청→매칭→주문→결제→정산). 기존 크몽 후기·구간3 진단 퍼널("사전 상담")을 흡수해 자체 마켓플레이스화. 브랜드 컬러(NAVY `#1B2B4B` / GREEN `#27AE60`) 정합.

**페이지:**
```
/showcase            ← 공개 마켓플레이스 메인 (히어로+매칭위젯, 5단계, 인기멘토, 후기 마퀴, 실시간 통계)
/showcase/mentors    ← 멘토 목록 (직무·서비스 필터)
/showcase/reviews    ← 후기 (크몽 임포트, 마스킹)
/showcase/faq        ← FAQ
/showcase/apply      ← 멘토 지원(가입) 폼
/mentor/login        ← 멘토 로그인
/mentor              ← 멘토 포털 (프로필 편집·서비스·받은 신청 수락/거절)
```

**API 라우트:**
```
GET  /api/showcase/stats           ← 합격수·진단수·진행중·후기·평점 집계
GET  /api/showcase/feed            ← activity_feed (마스킹 실시간 활동)
GET  /api/showcase/mentors         ← 공개 멘토 목록 (approved & visible)
GET  /api/showcase/reviews         ← 후기 목록
POST /api/showcase/mentor-apply    ← 멘토 지원 접수 → mentor_applications
POST /api/showcase/mentor-interest ← 멘티 관심/신청 → mentee_applications
POST /api/mentor/login · logout    ← 멘토 인증 (password_hash)
GET  /api/mentor/me                ← 본인 프로필+서비스
GET  /api/mentor/requests · [id]   ← 받은 멘티 신청 조회 / 수락·거절 → matches
```
> ⚠️ **middleware 확인 필요:** `/api/*`는 인증 게이트 대상. 공개용 `/api/showcase/*`·`/api/mentor/login`이 `PUBLIC_API`에 등록 안 되면 비로그인 401 → 쇼케이스 작동 불가.

**Supabase 테이블 (신규 11개 — 🔒 FROZEN 무수정, diagnoses/mentees는 FK 읽기 조인만):**
```
mentor_profiles      — 공개/심사용 멘토 프로필 (status, is_visible, 로그인 email·password_hash·last_login_at, 평점 캐시)
mentor_applications  — 멘토 지원(가입) → 승인 시 mentor_profiles 생성
services             — 멘토별 상품 (크몽 product 매핑, price_band)
reviews              — 크몽 임포트 후기 (정규식 마스킹, LLM 0토큰)
mentee_applications  — 멘티 신청 (desired_mentor_id, diagnosis_id = 🔒읽기 FK only)
matches / match_events — 신청→수락/거절→진행/완료 + 상태 감사 이력
orders / payments / settlements — 결제·에스크로·정산 (Phase 3 골격, fee nullable=수익모델 미정)
activity_feed        — 쇼케이스 실시간 활동 피드 소스
```

**신규 SQL (실행 순서):** `supabase-marketplace-setup.sql`(테이블 10) → `mentor-apply.sql`(mentor_applications) → `mentor-auth.sql`(로그인 컬럼) → `seed.sql`(시드)

**핵심 파일:** `app/showcase/ui.tsx`(공용 UI — 카드·모달·후기 마퀴·슬라이더), `app/showcase/page.tsx`, `app/mentor/page.tsx`

**멀티멘토 대비:** 전 테이블에 `mentor_id` FK 선반영. 초기엔 영석님 1인 시드.

---

### 🚧 모듈 13 — 멘토링 운영(딜리버리) — 2단계 BUILT (SQL 실행 완료 2026-06-13, lib/pdf.ts 복구 후 3단계)

> **상태:** ✅ 1단계 + S1 AI 보조 동작(회차 S0~S4 운영 + 그룹화 AI 제안). SQL 적용·E2E 검증 완료(여정 생성·회차 PATCH·이용기간 OT+3개월·Claude 그룹화·DB 저장). 피드백 반영: ① `makeup` 계약 있을 때만 노출 · ③ 회차 순차 잠금. 2~3단계·타 회차 AI 미착수. **계약 문서: 루트 `PLAN.md`**.
> **SoT:** `admin-tool/mentoring tool/files/멘토링_운영표준_SOP_v1.md`(SOP v1.1) · `멘토_운영스크립트_v1.md`(v1.0)

**역할:** 결제(makeup 계약) 이후 **5세션(S0~S4) 멘토링 실행**을 운영. 회차 진행·과제 게이트·재량 판단 기록 →
**녹취→STT→8섹션 리포트→멘토 검수→PDF**(SOP §2.4 자동화 1순위) → 종료까지 케이스 라이프사이클 관리.
어드민의 "네 번째 축" — ①진단(FROZEN)·②마케팅·③마켓플레이스에 이은 **딜리버리 축**.

**상품 매핑 (확정 2026-06-11):** SOP "단일 OT+4회차=5세션" = `makeup`("회사선택→이력서 Make-up", 200,000원).
S0=OT(진단) / S1=타깃 그룹화 / S2=경험 구체화 / S3=심화과정 / S4=심화과정(추가). 나머지 5종은 단발 서비스(회차 비대상).

**신규 테이블 (🔒 모듈1 FROZEN 무수정 — mentees/contracts는 FK 읽기조인만):**
```
case_journeys    — makeup 계약 1건 = 멘토링 여정 (track 신입/경력, deadline=OT+3개월, 휴면/긴급)
case_sessions    — 회차 S0~S4 (status, 과제 게이트, recording_url, stt_text, judgment_note 재량 1줄)
session_reports  — 표준 8섹션 리포트 (sections JSONB, status draft/reviewed/sent, pdf_url)
case_artifacts   — 작업 산출물 (target_groups·appeal_tags·representative_experiences·placeholders, 전부 JSONB)
```
> 설계 원칙(SOP §0·§6): 절차·산출물은 구조화 / 판단은 재량+근거기록 / **6축 점수 멘토 비개입(보정 필드 없음)** / 어필태그 개방형.

**단계 로드맵 (상세 `PLAN.md`):**
```
0 ✅ 거버넌스 등재 (이 섹션 + PLAN.md)
1 ✅ 멘티 상세 회차(S0~S4) 탭 + 과제 게이트 + 재량 기록 — SQL + journey/sessions API + JourneyPanel.tsx. tsc 통과. ⚠️ SQL 실행 대기
2    녹취→STT(클로바/Whisper)→8섹션 리포트(Claude)→검수→PDF. ⚠️ lib/pdf.ts throw 복구 선행 (자동화 1순위)
3    멘토 포털에 운영 스크립트·S0 체크리스트·메시지카드9·실수Top7 + 예외처리(긴급/휴면/리스크) 상태머신
```

**1단계 산출물 (2026-06-11):** `supabase-mentoring-ops-setup.sql`(case_journeys·case_sessions) · `app/api/mentees/[id]/journey/route.ts`(GET·POST 5회차 시드·PATCH) · `.../journey/sessions/[sid]/route.ts`(PATCH) · `app/admin/mentees/[id]/JourneyPanel.tsx`(page.tsx 삽입). SESSION_BLUEPRINT(S0=무준비·S1~S4 과제전제)는 journey route 상수.

**UI 개선 (피드백 반영):** ① `makeup` 계약이 있을 때만 멘토링 진행 섹션 노출(없으면 숨김, 기존 여정 있으면 보존) · ③ 회차 순차 잠금(이전 회차 `done` 전까지 🔒 — 상태변경·펼침·편집 비활성, S0 완료→S1 해제…).

**AI 보조 레이어 (피드백②, S1 그룹화 1차 — 2026-06-11):** `lib/mentoring/group-suggest.ts`(Claude sonnet-4-6 — 공고목록 → 직무 테마 그룹·제외사유·직무개념 교정 **제안**, 멘토 검수) · `.../sessions/[sid]/suggest-groups/route.ts`(POST) · S1 회차 카드 UI. `case_sessions`에 `ai_suggestion`(JSONB)·`assignment_input`(TEXT) 추가. 실 Claude 호출 E2E 검증 완료(SOP S1 로직 재현: 테마 묶기·"생산관리 vs 공정품질" 교정·커리어자산 제외). ⚠️ 공고명 원문 보존 6/10(모델 특성 — 후속 튜닝 대상). 설계원칙: **AI는 제안만, 점수·판단 비개입**(SOP §0).

**AI 보조 — S2 경험 합리화 (2026-06-11):** `lib/mentoring/experience-suggest.ts` + `.../sessions/[sid]/suggest-experience/route.ts`(**S1 그룹 자동 참조** → 그 군 언어로 번역) + S2 카드 UI. 합리화 번역(experience 원문→after)·과대표현 정직화·리스크 소재(종교 등)·대표경험 후보. `ai_suggestion` 컬럼 공용(추가 SQL 없음). **환각 해결**: 입력을 줄단위 항목으로 번호 매기고 `experience`를 **코드가 입력 원문으로 강제 교체**(ref 매핑) → 창작·누락 0(6/6 원문 일치 검증, 종교 리스크 분류 포함). 긴 입력 시 출력 잘림 → ① **잘림 복구**(`repairTruncatedJson`: 잘려도 완료 항목까지 살려 502 방지) ② **experience 모델 생성 생략**(빈 문자열, 코드가 ref로 원문 복원 → 출력 토큰 절반) + max_tokens 8000. **20항목 48초·원문 100%·창작/누락 0·502 없음** 검증. 설계: AI는 번역·플래그만, experience 원문 보존·점수 비개입(SOP §0).

**✅ 2단계 — 멘티 전달 리포트 (2026-06-13):** 회차 AI 결과 → **멘티용 리포트 생성**(`lib/mentoring/report-generator.ts` sonnet-4-6 — headline·맥락·임팩트·highlights·액션, SOP §5 멘티 톤 + **현직 인사담당자 시각**(채용담당자 평가 언어)·합니다체 공감형) → **react-pdf PDF**(`report-pdf.tsx`, Noto Sans KR ttf 9종) → **다운로드/메일**(Resend 첨부 — `noreply@rocketcareer.co.kr`). API: `.../report/route.ts`(POST 생성·PATCH 편집) · `report/pdf`(GET) · `report/send`(POST). `case_sessions`에 `mentee_report`(JSONB)·`report_status`·`report_sent_at` 추가. UI: SessionCard 리포트 영역(생성·미리보기·PDF·메일). ⚠️ 기존 `lib/pdf.ts`(throw)는 진단리포트(모듈11)용 — **무관**(react-pdf 신규 경로). dev 서버는 무거운 AI+react-pdf 연속 호출에 OOM 가능 → `scripts/dev.mjs`로 8GB(프로덕션 무관).

**📐 S1/S2 리포트 구조 재설계 (2026-06-13):** 단순 줄글 highlights에서 세션별 특화 구조로 전면 개선.
- **S1** = `S1GroupCard[]`(`group_name`·`fit_summary`·`appeal_points[]`·`approach_tips[]`·`hr_read` — 그룹별 접근전략 카드). highlights 대신 그룹카드 렌더.
- **S2** = `S2ExperienceMapping[]`(`experience` × `per_group[group_name·positioning·key_message]` — 동일경험이 그룹별로 어떻게 다르게 어필되는지 매트릭스). report/route.ts가 S2 생성 시 S1 `ai_suggestion` 자동조회 → `s1Context`로 전달(그룹명 참조).
- **전 회차** = `RoadmapInfo`(step 1~5 — 코드가 삽입, AI 비생성). PDF·UI 모두 5단계 인디케이터 렌더.
- `report-pdf.tsx`에 `Roadmap`·`GroupCards`·`ExperienceMatrix` 컴포넌트 추가. `JourneyPanel.tsx` UI 미리보기 동기화. tsc 에러 0.

**🔧 리포트 버그픽스·표현 개선 (2026-06-13):** ① S2 JSON 잘림 해소 — `repairTruncatedJson` 추가 + max_tokens 4000→6000(S2). ② `parseJson`에서 가운뎃점(·) → 쉼표+공백(, ) 전환 후처리. ③ 로드맵 레이블: S2=경험 구체화 / S3=심화과정 / S4=심화과정(추가) (`report-pdf.tsx`·`report-generator.ts` 양쪽 동기화).

**🎙 STT 파이프라인 구현 (2026-06-14):** 녹취→STT→리포트 자동화 완성. `lib/mentoring/stt.ts`(Google STT V1 REST + ffmpeg 5분 청크 분할·JWT 인증 tts-generator.ts 동일 패턴 재사용) · `app/api/mentees/[id]/journey/sessions/[sid]/stt/route.ts`(PATCH=Track A 텍스트 직접 저장 · POST=Track B 파일 업로드→Supabase Storage→STT→DB · maxDuration=300) · `report-generator.ts`(sttText 6번째 인자, 프롬프트 상단 `[녹취 STT 내용]` 주입, max_tokens sttText 있으면 5000) · `report/route.ts`(stt_text SELECT·전달) · `JourneyPanel.tsx`(Session 인터페이스 recording_url/stt_text 추가, STT 섹션 UI Track A+B, genReport 버튼 "STT 포함" 배지). Google STT: `encoding:MP3·sampleRateHertz:16000·languageCode:ko-KR·model:latest_long·enableAutomaticPunctuation:true`. 인증 환경변수 재사용(GOOGLE_SERVICE_ACCOUNT_EMAIL/GOOGLE_PRIVATE_KEY). tsc 에러 0.

**재사용 자산:** `app/admin/mentees/[id]/page.tsx`(계약·메모·파일 그릇), `STAGE_TEMPLATES`(`app/api/mentees/[id]/contracts/route.ts` — makeup 단발 stages를 회차로 대체), `app/mentor/`(모듈12 포털, 3단계 스크립트 임베드).

**🆕 멘티 포털 (멘티 화면 — Phase A~C, 2026-06-18, 코드완료·tsc 0·SQL 미적용):** 멘티가 직접 로그인해 회차별 자료를 제출·리포트를 열람하는 **멘토 화면(JourneyPanel)의 멘티 카운터파트**.
- **인증(매직링크·passwordless):** `lib/auth.ts` `createMenteeToken/verifyMenteeToken`(`mentee_auth` 7일) + `createMenteeLink/verifyMenteeLink`(매직링크 7일, `mlink:` 네임스페이스 → 세션 오용 차단). mentees 스키마 무수정(stateless). `middleware.ts` 멘티 게이트·`PUBLIC_API`(login/verify/logout)·matcher `/mentee/:path*`.
- **라우트:** `app/api/mentee/{login,verify,logout,me}` + `sessions/[sid]`(PATCH 제출)·`sessions/[sid]/files`(업로드/서명URL)·`sessions/[sid]/report(+pdf)`(sent만, `renderReportPdf` 재사용). 공용 `lib/mentee-portal.ts`(`menteeIdFrom`·`safeSession` 노출 화이트리스트·`ownedSession` 소유권·`serializeSubmission`).
- **UI:** `app/mentee/{login,page}.tsx` + `forms.tsx`(S0 진단체크리스트·S1 회사선택 Roadmap·S2 경험시트(신입/경력)·S3·S4 업로드·파일 업로더·리포트 뷰·개인 TO-DO). 입력 항목 = SOP §3·운영스크립트 회차 명세.
- **데이터 흐름:** 멘티 제출 → `case_sessions.mentee_submission`(JSONB, 신규) + `assignment_input`(직렬화 텍스트, 기존 — group/experience-suggest 포맷 일치) + `assignment_done` + `submitted_at`. 멘토 `JourneyPanel`이 "멘티 초대" 버튼·"멘티 제출됨" 배지로 자동 표시, `aiInput`이 `assignment_input` prefill → 기존 AI 그룹화/합리화 **무수정**. 첨부 = `files`(신규 `session_id`)·`mentee-files`(private) 버킷.
- **보안:** 전 라우트 소유권 검증(token menteeId === journey.mentee_id), `safeSession`이 judgment_note·ai_suggestion·stt_text·recording_url 비노출·mentee_report는 sent만. 🔒 FROZEN 읽기조인만.
- **SQL:** `supabase-mentee-portal.sql`(case_sessions.mentee_submission/submitted_at·files.session_id). ⚠️ 미적용 시 제출 500. **후속(사장 승인):** SQL 적용·라이브 스모크·매직링크 메일 발송.

---

### ✅ 모듈 14 — AI 운영체계 (자율 협업 오케스트레이션)

> **역할:** 사장/커리어랩장의 오더를 받아 **영향받는 부서만 자동 연결** → 협업·실행·교차검수·정합·보고까지 구동하는 메타 레이어. 코드 모듈(1~13)의 **상위 운영 거버넌스**.
> **트리거:** 하이브리드 트리아지(`CLAUDE.md` 최상단) → 풀 프로토콜은 `/order`.

**구성 (`org/`):**
```
PROTOCOL.md    — 10단계 오더 인테이크 절차 (트리아지→영향판정→부서선별→협업→실행→교차검수→정합→보고→피드백)
IMPACT_MAP.md  — 영향범위 라우팅 테이블 (이 ARCHITECTURE 의존성·변경임팩트·IMC 결정화)
CHARTER.md     — 객관성 헌장·증거 등급·자율성 경계·리스크 레지스트리·교차검수 페어링·금지 원칙
OBJECTIVES.md  — 회사 목표 (커리어랩장 소유)
LEDGER.md      — 피드백 원장 (분신 소유, 지시→진행→완료→검증, 근거 없이 검증 불가)
ORG_CHART.md   — 부서·에이전트 R/R 맵 (경영지원실 소유)
state/*.md     — 6본부 상태파일 (현황·목표정합·블로커·미반영피드백)
```

**에이전트 (`.claude/agents/`):** `chief-of-staff`(분신)·`career-lab-lead`(커리어랩장) + 6 본부장(`rnd-head`·`marketing-head`·`bizdev-head`·`sales-head`·`it-security-head`·`management-support-head`). 본부장은 분석·제안 전담, 실행은 분신이 코드 에이전트(계층3, 기존 6종) 위임.

**커맨드:** `/order`(인테이크) · `/standup`(일일·분신) · `/strategy-review`(주간·커리어랩장) · `/org-audit`(객관성 검증).

**운영 콘솔 (회사 운영 콕핏 — dev 전용, 2026-06-13):** 위 org/ 파일을 GUI로 보고 편집하는 한 겹.
```
admin-tool/lib/org/*.ts        — 파서/직렬화 (registry·markdown·ledger·agents·objectives·state·orgchart)
admin-tool/app/api/org/*       — overview·agents/[name]·ledger(+[id])·objectives·state/[dept]·orgchart (전부 prod 403 가드)
admin-tool/app/admin/org/      — 3탭(조직 맵·오더 보드·목표) + 부서 드로어(설정·오더·현황·협업)
```
- **파일=단일 진실원천**: Node fs로 `org/*.md`·`.claude/agents/*-head.md` 직접 read/write. 대시보드 편집 = Claude Code가 읽는 그 파일. **분리형**(설정=콘솔 / 실행=`/standup`·`/order`).
- **안전**: 쓰기는 org/+경영 8 에이전트만. LEDGER append-only(POST 추가·PATCH 상태/근거 셀만). 고위험 키워드(CHARTER §4) 오더 자동 `승인대기`. admin-tool 코드·FROZEN·Supabase 미접촉.
- **검증(2026-06-13)**: `scripts/verify-org.mts` — 실제 파일 read/write 라운드트립(에이전트 무손실 바이트 동일·append-only 보존·고위험 게이트). tsc 0.
- Phase 2(배포): org 데이터 Supabase 이전 + `/org-sync` 브리지 (추후).

> ⚠️ **정합:** `IMPACT_MAP.md`는 이 ARCHITECTURE.md(의존성 맵·변경 임팩트 체크표·📣 마케팅 레이어)의 결정화다. ARCHITECTURE가 바뀌면 `IMPACT_MAP.md`도 동기화한다(`/sync-arch` 점검 대상). 운영체계는 FROZEN(모듈1)을 읽기조차 하지 않는다.

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
[대시보드]  ──► lead_analytics (✅ 트리거 LIVE 2026-06-13)

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
| case_* 신규 테이블 | 멘토링 운영(13) | mentees/contracts FK 읽기조인만, 모듈1 FROZEN 무수정 |

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

> 🆕 **멘티 인증 (2026-06-18):** `mentee_auth` 쿠키(HMAC, 7일) — 비밀번호 없이 **이메일 매직링크**로 발급(`createMenteeLink` 7일·`mlink:` 네임스페이스 → `/api/mentee/verify`에서 세션 토큰으로 교환). mentees 스키마 무수정(stateless). 멘티 라우트는 미들웨어 게이트 + 앱 레이어 소유권 검증(token menteeId === journey.mentee_id) 이중 방어. 공개: `/api/mentee/{login,verify,logout}`.

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
RESEND_FROM_EMAIL  ← 발신 주소 (현재: noreply@rocketcareer.co.kr — 도메인 검증 완료, 2026-06-13)
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
| `/sync-arch` | ARCHITECTURE.md ↔ 코드 **+ 마케팅 IMC 전략** 동기화 검증 + WORKLOG 기록 |
| `/casefile-script` | 케이스 → CaseFileVideo 대본(props+TTS) 생성 (10룰) |
| `/casefile-qa` | 생성 대본 3관점 자동 검수 (논리/HR언어/몰입) |
| `/shorts-strategy` | 쇼츠 콘텐츠 전략 (벤치마크·포맷·훅·CTA) |
| `/video-production` | 영상 자동생성 연구·결정 현황 |
| `/project-state` | 프로젝트 현황 요약 |
| `/order` | 오더 인테이크 — 영향범위 판정·부서 선별·교차검수·두괄식 보고 (운영체계) |
| `/standup` | 일일 운영 점검 (분신) — 본부 현황 종합·LEDGER 추진 |
| `/strategy-review` | 주간 목표 점검 (커리어랩장) — OBJECTIVES 갱신·개선 지시 |
| `/org-audit` | 객관성 검증 — LEDGER 완료/검증 주장을 실제 근거와 대조 |

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
| ✅ 완료 | 마켓플레이스 SQL 4종 실행 (setup→apply→auth→seed) | 마켓플레이스(12) | 2026-06-07 Supabase 실행 완료 (11개 테이블·멘토 로그인 컬럼·시드) |
| ✅ 완료 | 마켓플레이스 main 머지·배포 | 마켓플레이스(12) | 2026-06-07 `feature/marketplace-phase1`→main FF 머지·푸시(`0abd512`)·Vercel 배포. PUBLIC_API(showcase·mentor) 등록 확인됨 |
| 🟡 중간 | 멘토 인증 이원화(자체 password_hash ↔ 모듈10 Supabase Auth) 통합 방향 | 마켓플레이스·Phase C | IMC v2 §1 'Track B' 정합은 완료(2026-06-07). 모듈 10·12 인증 통합 방향만 미정 |
| 🔴 높음 | **모듈13 1단계 SQL 실행**: `supabase-mentoring-ops-setup.sql` Supabase 적용 (코드 완료·tsc 통과, 미실행 시 회차 API 500) | 멘토링 운영(13) | case_journeys·case_sessions. makeup 계약 연결. FROZEN 무수정 |
| ✅ 완료 | **모듈13 2단계 완료 (2026-06-14)**: STT 파이프라인(Google STT + ffmpeg 5분 청크·2트랙) + sttText→리포트 자동 반영 + JourneyPanel STT UI | 멘토링 운영(13) | stt_text 있으면 리포트 AI 프롬프트에 자동 주입. 3단계(멘토 포털) 미착수 |
| 🟢 낮음 | **모듈13 3단계**: 멘토 스크립트·체크리스트·메시지카드9 + 예외처리 상태머신 | 멘토링 운영(13) | 모듈12 멘토 포털 연동 |
