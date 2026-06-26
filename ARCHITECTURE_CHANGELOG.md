# ARCHITECTURE 변경 이력 (CHANGELOG)

> ARCHITECTURE.md '마지막 업데이트' 변경 이력 전체 보관소. 최신이 맨 위.
> /sync-arch가 새 항목을 이 파일 맨 위에 추가하고, ARCHITECTURE.md에는 최근 3건 요약만 둔다.
> 평상시 작업/sync에서 이 파일을 통째로 읽지 말 것(토큰 절약). 특정 이력 조회 시에만 Grep/부분 Read.

---

## 2026-06-26 (**어드민 상품관리 ↔ 랜딩 상품카드 정합** — `/admin/products`(products 테이블)가 메인 사이트 랜딩 상품 카드와 매칭 안 됨. 진단: ① 랜딩 동기화가 **가격만** 덮어써 어드민 제목·소개 편집이 사이트에 무반영, ② 어드민 제목('경력기술서 작성')과 카드 제목('연봉·경력 더 인정 받는 경력기술서 Value-up') 드리프트로 식별 불가, ③ 랜딩 카드 6개 중 '대기업 서류+면접 자소서'(90,000)는 `data-product-key` 없어 관리 불가·어드민 `package`(188,000)는 대응 카드 없는 고아. 결정(사장): 범위=제목+소개+가격 전체 / 현재 어드민 가격이 정답 / 미매칭 카드는 새 키 등록. 조치: ① `public/landing.html` 동기화 JS 확장 — 가격에 더해 `.service-title`·`.service-desc`를 `el.closest('.service-card')`로 찾아 DB title·intro로 치환(실패 시 정적 폴백), 카드#3에 `data-product-key="package-doc"` 추가, 정적 폴백가 정합(makeup 170k·coverletter 88k·salary 50k). ② DB 정합(사장 승인, service-role) — 5개 기존 상품 title·intro를 랜딩 카피로, interview price null→109,000, sort_order 시각순, `package-doc` 신규(90,000), `package` is_public=false. `supabase-products-sync.sql`(멱등)·`supabase-products.sql`(시드) 동기 반영. ③ 어드민 안내문 갱신(사이트 실시간 반영·시작가 명시). 결과: 공개 상품 6개 ↔ 랜딩 카드 6:6 1:1, 어드민이 카드 제목·소개·가격 단일원천. preview 검증 통과(콘솔 에러 0). 🔒 FROZEN·가격 외 변경 없음)

---

## 2026-06-25 (**sync-arch 토큰 최적화 — 변경 이력 아카이브 분리** — sync마다 토큰 폭증(ARCHITECTURE.md `마지막 업데이트` 1줄이 ~1.4만 자·27건 누적, 작업 전 필수 읽기라 전 작업에 과금) 해소. ① ARCHITECTURE.md 변경이력을 `ARCHITECTURE_CHANGELOG.md`로 분리 — 본문엔 `마지막 업데이트` 1줄 + `최근 변경` 3건 요약만(line7 13,958→248자·파일 65.8K→50.3K). ② WORKLOG.md 43블록 → 최근 5개만 유지, 38블록 `WORKLOG_ARCHIVE.md`로 이동(47.6K→5.2K). ③ `/sync-arch` 스킬 갱신: 토큰 절약 원칙(아카이브 비독·ARCHITECTURE/WORKLOG는 head·limit·Grep로만)·CHANGELOG 분리 절차·WORKLOG 로테이션 스크립트(UTF-8 no BOM, `Get-Content` ANSI 함정 주석). 데이터 손실 0(git·아카이브 보존). 🔒 FROZEN·코드 미접촉)

---

## 2026-06-25 (**아침 시장메일 동적화 + 랜딩↔어드민 상품가격 동기화** — ① 아침 시장조사 메일이 매일 비슷하던 문제: `diagnoseMarketPhase(date)`가 **월별 한국 채용시장 국면**(1~12월 캘린더)을 진단해 **검색 그리드가 시기별로 유동 변화**(고정 항목 X) + 전날 리포트 전달(미중복 지시) + finance-data 사업현황 믹스(인사이트). 새 메일 구조 📌인사이트→🗓️국면→🔍시기맞춤 동향→🧭조직 액션, 제목에 국면 표기. 웹검색 머리말·코드펜스(```html) 누출은 `cleanHtml`(첫 `<h3>`/📌부터) 후처리. prod 검증 200/30.4s·제목'하반기 계획·인턴십 시즌'·4마커·출처4. `lib/market-research`·`15110e9`·`22a5b82`. ② 랜딩 상품가격이 어드민과 불일치 → **동적 자동 동기화**(사장 선택): 공개 `GET /api/showcase/products`(`is_active&&is_public`만, 미들웨어 `/api/showcase` PUBLIC_API 자동) + 랜딩 카드 `data-product-key` 5종(resume·coverletter·makeup·interview·salary) + 로드시 fetch→가격 치환 스크립트(실패 시 정적 유지). 결과: 자소서 80→88·메이크업 160→170·연봉 40→50 자동교정 — **어드민 가격만 바꾸면 랜딩 반영**(단일원천). prod 검증 endpoint 6상품·키5·스크립트 deploy. `admin-tool/public/landing.html`·신규 `app/api/showcase/products`·`ef9dabd`. ⬜ 사장(어드민 상품): 면접 가격 입력(현재 null→랜딩 정적)·랜딩 '서류+면접 자소서'/어드민 `package` 대응상품 정리. 🔒 FROZEN 미접촉)

---

## 2026-06-25 (**랜딩+admin-tool 코드병합 1단계 + 오더리포트 JSON 복구** — ① 정적 랜딩(루트 `index.html`+`style.css`)을 admin-tool `public/landing.html`·`style.css`로 흡수 + `next.config` beforeFiles rewrite `/`→`/landing.html`(다른 경로 불변) + 랜딩 절대URL→상대경로(`/showcase`·`/api/submit-diagnosis`, 같은 origin). prod 검증(`/`=랜딩200·showcase200·admin307·css200). ⬜ 사장: 도메인 `rocketcareer.co.kr` 이전(Vercel). 2단계(React 변환)는 선택·`97f2772`. ② `org-orders`/`org-execute` `parseJson`에 절단복구(`repairTruncatedJson`)+토큰상향 — 복합 오더 'Unterminated string' 해소·`ea52c3f`. 🔒 FROZEN 미접촉)

---

## 2026-06-24 (**멘토 온보딩 규제 방어 5단계 — 용역계약/윤리서약 분리·개별 확인** — 알선 방어(교육만·연결 안 함) + 근로자성 방어(독립 프리랜서) 2선을 동의 절차에 반영. ① `legal`(용역계약: 독립 프리랜서·업무범위 한정·성과 비연동·사업소득 3.3% 필수문언)과 `pledge`(윤리서약: 알선금지·합격보장금지·비밀유지) **단계 분리** → 독립 증거. ② pledge의 알선금지·성과비연동을 **개별 체크박스**(둘 다 체크해야 서약 버튼 활성)·확인시각 기록(`brokerage_ack_at`·`outcome_pay_ack_at`). ③ 동의 페이지가 단계 **전문 표시**(사전 고지·검토). ④ 콘텐츠: 금지표현(근무·급여·지시·전속·성공보수) 배제 + '지시→범위·통제→합의·고용→용역' 프레이밍, edu1 업무범위=**권장 안내서**(의무 지시 아님), edu3 정산·세무(3.3%·종합소득세) 재반영. ⑤ **fix(auth)**: `verifyOnboardingToken` 단계 화이트리스트에 `pledge` 누락 → 토큰 거부 버그(②서약 링크 전면 차단) 수정. `lib/auth`·`lib/mentor-onboarding-{content,flow}`·`app/onboarding/agree`·`/api/onboarding/agree`·`supabase-mentor-onboarding.sql`(pledge·ack 컬럼)·`2395d9c`. tsc 0·build 0·5단계 렌더+체크박스 게이트 검증·main FF 배포. ⬜ 사장: SQL 적용(기록·게이트 활성)·노무사/세무사 1회 검토(초안 명시)·정산 방식(고정액/%) 확정 시 대가 문구 반영. 🔒 FROZEN 미접촉)

---

## 2026-06-23 (**오더 운영 루프 — 프로덕션 자율화(모듈14 v2)** — 오더 단일 진실원천(SoT)을 **Supabase `org_orders`/`org_order_events`** 로 단일화(웹·채팅 공용). 기존 분리(오더 보고=Storage advisory / 오더 보드=dev전용 파일 LEDGER / 실행=채팅뿐)를 하나의 프로덕션 루프로 통합. ① `/admin/orders` = **프로덕션 오더 콕핏**: 보고서 굴리기(수정오더/의견→분신 재분석 스레드 `revise`)·상태 칸반(지시~검증/반려)·액션 '오더 발행'(본부 배정 자식 `dispatch`). ② **채팅 단일화**: `scripts/orders-cli.mts`(`npm run orders`) + /order·/standup·/org-audit·PROTOCOL·IMPACT_MAP를 org_orders 기준 갱신, `org/LEDGER.md`는 동결 히스토리(L-0001~0019). ③ **서버사이드 자율 실행(3단계)**: `/api/orders/[id]/execute` — 배정 본부 페르소나(임베드 `lib/org-personas`)가 실행 + 교차검수(실행≠검수, 다른 부서)→`완료`/`반려`, **고위험·FROZEN은 자동 차단(승인대기)**. 천장: Vercel 60s·라이브툴 없음 → heavy/무인 멀티에이전트는 채팅/워커(3b 미래). `lib/org-orders`(Storage→테이블·risk게이트·스레드)·`lib/org-execute`·신규 API 4종(`[id]` PATCH·`revise`·`dispatch`·`execute`). OWNER_ONLY 자동 상속. tsc 0·build 0. ⬜ **사장 선행: `supabase-org-orders.sql` 적용**(미적용 시 graceful 빈 목록). 🔒 FROZEN 미접촉)

---

## 2026-06-23 (**상단 메뉴 대메뉴 그룹핑 + 오더보고 드롭다운 + 멘티 테스트계정** — ① 어드민 상단 네비를 5개 대메뉴(접수·멘티/멘토/마케팅/경영/설정) hover 소메뉴로 그룹핑(역할·dev 필터 유지)·`7c2295a`. ② '오더 보고' 이전오더 드롭다운(재분석). ③ 멘티 교육/동의 화면 확인용 테스트 멘티+여정+7일 매직링크(스크립트). 🔒 FROZEN 미접촉(테스트 멘티 1행=명시적 요청))

---

## 2026-06-22 (**멘티 동의 게이트 + 조직 결과리포트(오더→AI 구조화)** — ① 멘티 포털 첫 진입 개인정보 동의+'단계 프로그램' 안내 게이트(동의 전 대시보드 차단), `case_journeys.mentee_consented_at`·`eeff0c0`. ② 조직 결과리포트 `/admin/orders`(OWNER_ONLY): 기존 콘솔(dev전용·파일기반·오더 '지시'만 기록·피드백0)의 프로덕션 보완 — 오더 입력→분신(COO) AI가 현상·문제·원인→목표→자원→이해관계→액션플랜→다음한수 구조화→Supabase Storage(`org-orders`) 저장·열람. `lib/org-orders`·`/api/orders`·`8789133`. 🔒 FROZEN 미접촉)

---

## 2026-06-22 (**시장조사 504 해결(Haiku) + 멘토 온보딩 단계 게이트** — ① 아침 시장조사 웹검색이 Vercel 60초 초과로 504(cron·수동 실패) → 합성 모델 `claude-haiku-4-5`로 전환 + 시간예산/폴백, 실측 29.5초·출처13개 진짜 웹검색·`8b3c080`. ② 멘토 온보딩 단계 게이트: 법적동의→1~3차 교육, `/onboarding/agree`(공개·POST 기록·프리페치 방지)에서 단계 동의→다음 단계 자동발송, `mentor_onboarding` 테이블(IP/UA 감사·`supabase-mentor-onboarding.sql`), 완료 전 멘티배정 차단(여정 POST 게이트·테이블 미적용 시 OFF). `lib/mentor-onboarding-{content,flow}`·`lib/auth` monb:토큰·`702da21`. 콘텐츠 멘티='멘티님'/차분톤, 법적문서 템플릿(법률검토). 🔒 FROZEN 미접촉)

---

## 2026-06-22 (**시장리포트 누적 Storage 전환 + 멘토 온보딩 톤 + 세무 조직 신설** — ① #2 시장리포트 누적: `market_reports` 테이블 의존 제거 → Supabase Storage(`market-reports` 버킷 `index.json`, 서비스롤·DDL 불필요)·`saveMarketReport`/`listMarketReports`·`9fc071e`(SQL 삭제). 누적 미동작 원인=테이블 미적용이었음. ② #1 멘토 온보딩 메일 현직 인사담당자 톤 재작성·세무 안내 제거(세무 조직 정비 후 재반영). ③ #3 세무 조직 신설: `.claude/agents/tax-advisor.md`(경영지원실 산하 세무 전문위원, 한국 세법+사업구조 접지) + ORG_CHART·management-support-head·OBJECTIVES·IMPACT_MAP 동기화. 🔒 FROZEN 미접촉)

---

## 2026-06-21 (**시장 리포트 어드민 + 멘토 온보딩 + 메뉴 정비 (/order 8건 배치)** — ① #1·#2 시장조사 메일을 어드민 누적·열람 + 수동 발송: `lib/market-research`(크론·수동 공유·🧭부서별 시사점) · `/admin/market-research` · `/api/market-research/{run,list}` · `market_reports`(`supabase-market-reports.sql`)·`0084f37`. ② #7 네비 '재무'→'매출·재무'+'시장 리포트'. ③ #4 멘토 온보딩 메일 `lib/mentor-onboarding`·`/api/mentors/[id]/onboard`+멘토 상세 버튼(정산/세무 #6 포함)·`60344c0`. ④ #3(중간)·#5·#8 LEDGER L-0011~L-0016 순차화. ⚠️ **배포 블로커**: Vercel `CRON_SECRET` 한글값 → HTTP헤더 ASCII 위반 빌드실패(사장 ASCII 교체 후 반영, 프로덕션은 `6b51d74` 정상). 🔒 FROZEN 미접촉)

---

## 2026-06-21 (**S1→S2 회차 버그픽스 + 판매분석 툴 + 아침 시장조사 Cron 메일 (프로덕션 배포)** — ① 멘티 포털 S1 완료 시 S2 미활성 버그: 회차 PATCH가 `done` 전환 시 다음 회차(seq+1) `locked→assignment_pending` 연쇄 + 멘티 화면 잠금 순차전용 보정(레거시 즉시 해제)·`dd1e543`. ② 판매분석 `/admin/finance` 확장 — `SalesAnalysis`(현황분석 버튼→`/api/finance/commentary` 영업·사업개발 관점 AI 코멘트, 상품×월 매트릭스·필터) + `lib/finance-data` 집계헬퍼(`financeSummary`·`productMonthMatrix`)·`FINANCE_SALES`(상품×월, 원본 `.xls` 입수 시)·`/api/finance` OWNER_ONLY·`aa10e3b`. ③ 아침 시장조사 자동메일 `vercel.json` crons(08:00 KST)→`/api/cron/market-research`(CRON_SECRET 검증, 미들웨어 `/api/cron/` 공개)→Claude `web_search` 3C/4P + 내부 판매현황 → `sendMail(OWNER_COPY)`·`6b51d74`. 🔒 FROZEN `lib/anthropic` 미접촉(신규 클라이언트). tsc 0·next build 0. 사장 선행: Vercel `CRON_SECRET` 추가·크몽 매출 `.xls` 재첨부(상품×월))

---

## 2026-06-21 (**상품 카탈로그 + 재무 대시보드 + 멘토 노출/사진 편집 + 메일 정비 (프로덕션 배포)** — ① 상품 SoT `products` 테이블 + `/admin/products` CRUD(타이틀·소개·가격·세션·노출)·`a5fd1c7`(SQL `supabase-products.sql`). ② 재무 `/admin/finance`(**사장 전용 OWNER_ONLY**) — 크몽 수수료·광고 세금계산서 1~5월 집계 + AI 운용비 → 월별 손익·추이·상품별·세무, `lib/finance-data.ts` SoT(임베드)·`bc7da08`. ③ 멘토 상세 `is_visible`·photo·연차 사장 편집(`lib/mentors` 매핑). ④ 멘토 화면 `app/admin/mentees/[id]/SubmissionView.tsx`(멘티 제출 항목 구조화)·회차 첨부 열람·`6275ef5`. ⑤ 메일 링크 `APP_URL` 프로덕션 고정 + 멘티/멘토 메일 `OWNER_COPY` BCC — 🔒 FROZEN 진단메일(`submit-diagnosis`·`inbox/send-email`·`send-email`)은 **사장 승인** 하 링크/BCC만 최소수정(`30bc873`, CLAUDE.md 예외기록). RBAC 멘토차단 실측(어드민 307·API 401). org LEDGER L-0007~L-0010 /standup 처리)

---

## 2026-06-19 (**멘티 포털 후속 — 리포트 사전정보 반영·회차 명칭 통일·멘토 열람/편집·S1/S2 양식 보강 (프로덕션 배포·검증)** — ① 리포트 AI가 멘티 제출(`assignment_input`) + 멘토 소통(재량 판단·과제 안내)을 근거로 분석(S0·S3·S4 포함, STT 기존)·`6f4ed95`. ② 회차 명칭 SoT `lib/mentoring/session-labels.ts`(배지 OT/Session 1~4 · 명칭 진단/타깃 그룹화/경험 구체화/서류 구체화/보강 및 보완) 멘토·멘티·관리자·리포트·메일 공통 + JourneyPanel '멘티 제출 자료'(텍스트+첨부 서명URL)·`60b7fa9`. ③ 멘토 AI 리포트 직접 편집 `app/admin/mentees/[id]/ReportEditor.tsx`(PATCH `final`) + 회차별 첨부 열람 `app/api/mentees/[id]/journey/sessions/[sid]/files` + S1/S2 폼 보강(점수 제거·양식 2종 반영)·`64bfb63`. 전부 main FF 배포·prod 검증(매직링크→제출→직렬화). SQL 추가 없음)

---

## 2026-06-18 (**멘티 포털(멘티 화면) 신규 구축 — S0~S4 풀 딜리버리, Phase A~C 코드완료·tsc 0** — 멘토 화면(JourneyPanel)의 멘티 카운터파트 신설. 멘티가 **이메일 매직링크(passwordless·`mentee_auth`, mentees 스키마 무수정·stateless)**로 로그인해 **회차별 자료를 구조화 제출**(`mentee_submission` JSONB → `assignment_input` 직렬화 → 멘토 JourneyPanel·AI 그룹화/합리화 제안 **무수정 동작**)하고 **리포트를 열람**(sent만). 신규 `app/mentee/*`(login·dashboard·forms)·`app/api/mentee/*`(login/verify/logout/me·sessions/[sid] 제출·files·report+pdf)·`lib/mentee-portal.ts`(인증·소유권·노출 화이트리스트·직렬화)·`lib/auth.ts` 멘티 헬퍼·`middleware.ts` 멘티 게이트. 멘토측 글루: `JourneyPanel.tsx` "멘티 초대"(매직링크) + S1/S2 "멘티 제출됨" 배지 + `journey/invite`. 입력 명세 = SOP §3·운영스크립트 회차. 🔒 FROZEN(mentees/diagnoses/contracts) 읽기조인만 + 전 라우트 소유권 검증(token menteeId===journey.mentee_id) + `safeSession` 노출통제(judgment_note·ai_suggestion·stt_text 비노출). SQL `supabase-mentee-portal.sql`(case_sessions.mentee_submission/submitted_at·files.session_id, **미적용**). 후속(사장 승인): SQL 적용·라이브 스모크·메일발송)

---

## 2026-06-16 (**마켓플레이스 생애주기 재설계 P1·P2 (프로덕션 배포·검증)** — 멘토–멘티 데이터 파편화(멘토 `mentors`↔`mentor_profiles`, 멘티 `mentees`↔`mentee_applications`, 인증 3분할) 통합 시작. **P1 멘토 단일화(#1·#2)**: `/api/mentors`(GET·POST·[id])를 `mentor_profiles` 단일 풀로 전환 + `lib/mentors.ts` 정규화(expertise→specialties·headline→소속·approved→active). 멘토 심사 승인분이 멘토 관리·진단 멘토지정 드롭다운에 즉시 노출. 멘티 배정 0건이라 mentor_id 재매핑 불필요. 프로덕션 검증: /api/mentors 1→7명 전환. **P2 로그인·권한 통합(#3)**: 통합 `/login`(사장 PIN·스태프/멘토 email+pw 단일 진입·역할 라우팅), `/staff-login`·`/mentor/login`→`/login` 리다이렉트, `/api/auth/me` 멘토 역할 추가+공개. 사장 PIN·미들웨어 owner 분기 무변경→잠금0(프로덕션 사장 로그인 200 확인). 로컬 E2E: 사장 me=owner·스태프 허용/사장전용307·멘토 me=mentor·포털200·/admin307. **후속**: P3(멘티 진단 단일 퍼널·case 자동생성)·P4(소통·파일) 미착수. ⚠️ 로컬 dev `.next` Windows 손상 반복 → 검증은 실데이터·프로덕션 우회. 커밋: admin-tool `fdfcad3`(P1)·`9ebd99e`(P2))

---

## 2026-06-14 (**RBAC 권한 3층 + 보안 강화 + 멘토 가입 동의 + 진단/리포트 개선 (프로덕션 배포·검증)** — ① **#6 RBAC**: 사장(PIN, 무변경)/운영스태프(`staff_users`·`/staff-login`·`staff_auth`)/멘토 3층. `middleware`가 사장전용(조직운영·멘토관리·멘토심사·스태프관리)만 스태프 차단(denylist `lib/permissions.ts`), 네비 역할필터. `/admin/staff`(사장)·`/api/staff`. E2E: 사장 전체200·스태프 허용200/사장전용307·API403. 프로덕션 사장 로그인 무손상 확인. ② **보안**: 전 public 테이블 RLS on(service_role 우회·무중단)+`mentee-files` 버킷 private+STT 녹취 경로저장(public URL 제거). ③ **멘토 가입**: `/showcase/apply`에 PII 수집·이용 동의 필수(`mentor_applications.consent_*`)—동의 미전송으로 막혔던 신청 정상화. 초대링크 `/showcase/apply` 통일. ④ **진단 리포트**: AI 추천상품 STEP4 자동체크·선택 반영 재생성, 추천서비스 박스 제거(본문 솔루션 중첩·🔒메일 사장승인 수정), 카카오 문의 버튼(/report 페이지). ⑤ **멘토링 리포트 메일** 카드 레이아웃 재설계+카카오(이메일·PDF). ⑥ **조직 콘솔** 협업 R/R 로그 표시·즉시반영 수정. ⑦ **랜딩** 멘토 매칭 보조 링크. 커밋: admin-tool `54de08e`·`b243e0e`·`1f397a3` / landing `488cab1`)

---

## 2026-06-14 (**모듈13 STT 파이프라인 구현** — Google STT + ffmpeg 5분 청크 분할, 2트랙(A=텍스트 직접입력·B=파일 업로드 자동변환). `lib/mentoring/stt.ts`(JWT 인증·청크 순차 STT 신규)·`stt/route.ts`(PATCH Track A · POST Track B · maxDuration=300 신규)·`report-generator.ts`(sttText 6번째 인자·프롬프트 상단 주입·max_tokens 5000)·`report/route.ts`(stt_text 조회·전달)·`JourneyPanel.tsx`(STT 섹션 UI · Session 인터페이스 recording_url/stt_text 추가). tsc 에러 0. 커밋: 미커밋)

---

## 2026-06-13 (**모듈13·14 프로덕션 배포** — admin-tool `5d3e1d0` → main FF 푸시로 Vercel Production 빌드 ● Ready(3m). 배포: 모듈13(멘토링 딜리버리·리포트, 내부 어드민) + 모듈14 콘솔(prod 숨김 — 네비 `devOnly` 필터·페이지 dev 전용 안내·API 403) + casefile 수정(`a33ec1c`). 전부 admin 인증 뒤·고객 노출 0·🔒 FROZEN 무수정. 콘솔 풀스택 검증(로그인 200·overview 8부서·agents 6섹션·page 200) 통과)

---

## 2026-06-13 (**운영 콘솔(회사 운영 콕핏) 구축** — 모듈14 org/ 파일을 GUI로 보고 편집하는 dev 전용 대시보드. `admin-tool/lib/org/`(파서 7) + `app/api/org/`(라우트 7, prod 403 가드) + `app/admin/org/`(조직 맵·오더 보드·목표 + 부서 드로어). 파일=단일 진실원천(Node fs로 org/·.claude/agents 직접 r/w), 분리형(설정=콘솔/실행=/standup). LEDGER append-only·고위험 자동 승인대기. `scripts/verify-org.mts` 실파일 라운드트립 검증(에이전트 무손실·append-only 보존·고위험 게이트), tsc 0)

---

## 2026-06-13 (**리포트 버그픽스·표현 개선** — `repairTruncatedJson`+max_tokens 6000으로 S2 JSON 잘림 502 해소. `parseJson`에 가운뎃점(·)→쉼표+공백 후처리. 로드맵 레이블 변경: S2=경험 구체화/S3=심화과정/S4=심화과정(추가). `report-pdf.tsx`·`report-generator.ts` 동기화)

---

## 2026-06-13 (**AI 운영체계(모듈14) 구축 + SQL 마이그레이션(L-0001) 완료 + 퍼널 측정 체계 + 랜딩페이지 수정** — org/ 파일 7종·에이전트 8개·커맨드 4개 신설. case_journeys·case_sessions·lead_analytics·bitly_clicks·trend_benchmarks·content_priority DB 실행 완료. lead_analytics 트리거 LIVE(EXCEPTION WHEN OTHERS 하드닝, diagnoses FROZEN 무손상). 랜딩페이지 v2: no-cors 제거·concern 필드 수정·URL 파라미터 추적 추가(bit.ly/rcl-shorts → ?ref=youtube_shorts → source_channel 자동 매핑). Bitly 토큰 등록 완료. LEDGER L-0001 검증)

---

## 2026-06-13 (**S1/S2 리포트 구조 재설계** — `S1GroupCard`·`S2ExperienceMapping`·`RoadmapInfo` 신규 인터페이스. S1=그룹별 접근전략 카드, S2=경험×그룹 포지셔닝 매트릭스, 전 회차=5단계 로드맵 인디케이터. S2 리포트 생성 시 S1 ai_suggestion 자동참조. `RESEND_FROM_EMAIL=noreply@rocketcareer.co.kr` 반영. tsc 에러 0)

---

## 2026-06-11 (**모듈 13 멘토링 운영(딜리버리) 시스템 등재** — 두 운영 문서(SOP v1.1·운영스크립트 v1.0, `admin-tool/mentoring tool/files/`) 평가 → 어드민의 "네 번째 축"인 멘토링 실행(딜리버리)이 비어있음 확인(녹취→STT→리포트 **코드 0건**, `lib/pdf.ts` generatePdf **throw 비활성**). 모듈 13 신규 등재(📋 PLANNED). **상품모델 결정**: `makeup`="회사선택→이력서 Make-up"(200,000원)=SOP 5세션(S0 OT~S4). **FROZEN 우회**: 회차·리포트는 신규 테이블(case_journeys/case_sessions/session_reports/case_artifacts) 분리, 모듈1(mentees/contracts/session_notes) 무수정·FK 읽기만. 계약 문서: 루트 `PLAN.md`)

---

## 2026-06-07 (**모듈 12 마켓플레이스 프로덕션 배포** — admin-tool `feature/marketplace-phase1` → main FF 머지·푸시(`0abd512`)로 Vercel 자동배포 + 신규 SQL 4종(setup→apply→auth→seed) Supabase 실행 완료(11테이블·멘토 로그인 컬럼·시드). 모듈 12 🚧 BUILT·미배포 → ✅ ACTIVE. 미결 'SQL 미실행'·'머지/배포 결정' 해소. IMC v2 §1 'Track B 미운영' → 'Phase 1 라이브'로 정합. ⚠️ main 직접 푸시가 auto모드 가드레일에 막혀 사용자가 `Bash(git push:*)` 권한 부여 후 단독 푸시로 진행)

---

## 2026-06-05 (**모듈 12 멘토링 마켓플레이스 등재 — 거버넌스 표류 시정** — `feature/marketplace-phase1`에만 있어 그간 미등재였던 양면 마켓플레이스(공개 쇼케이스+멘토 온보딩+멘토 포털+거래, 신규 테이블 11개)를 모듈 12로 인벤토리·상세·미결SQL에 등재. ⚠️ IMC v2 'Track B 미운영'·모듈 10 '멘토포털 미시작'과 상태 모순 표기. 미배포(main 미머지)·SQL 4종 미실행. 미머지 브랜치를 `/sync-arch`가 못 보던 맹점 노출)

---

## 2026-06-03 (**마케팅 전략 레이어(IMC) 거버넌스 등재** — `marketing_division/로켓커리어연구소_IMC_마케팅기획안_v2.md`를 전략 SoT로 "📣 마케팅 전략 레이어" 섹션 신설(전략결정→코드 매핑·정합 규칙), `/sync-arch`에 마케팅 정합 절차·출력 라인 추가)

---

## 2026-05-31 (모듈 8 인코딩 파이프라인 로직 박제: 표 추출 추가·max_tokens 16000·실패유형 매핑·왜인코딩하는지 3개 다운스트림 명문화. cases_career 29건/rules 173건/임베딩 29건 갱신)

---

## 2026-05-30 (CaseFile 영상 **완전 로컬 렌더 전환** — `scripts/make-case.mts`+`npm run make:case` / QA 게이트(`failOnQA`) / 폰트 weight 제한 최적화 / 어드민 웹 영상 UI 제거(쇼츠 버튼·롱폼 컬럼·케이스파일 웹 버튼) / **로컬 웹 패널 `/admin/local-studio` 신설**(dev 전용 — 케이스선택→대본생성→편집→로컬렌더→미리보기). **로컬 스튜디오 제작 이력 영속화**(renders API+배지+미리보기 복원) / **CTA 톤 개편**(첨삭 직접언급→커리어 진단/점검 3톤 변주·짧게) / **로컬 스튜디오 나레이션 우선 분리 플로우**(나레이션→자막→음성→렌더, QA 권고 대본 반영, caseId 기준 충돌 차단, 미리보기 버그·dev 안정화 수정) / **유튜브 업로드 키트**(렌더 후 ⑦ — 제목3·설명·랜딩 후킹 고정댓글))

---

