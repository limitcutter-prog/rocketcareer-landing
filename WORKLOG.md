# 작업 로그 (WORKLOG)

> /sync-arch 실행 시 자동 누적. 최신 항목이 맨 위. **최근 5개 블록만 유지**(초과분 → `WORKLOG_ARCHIVE.md`).
> 각 블록 = 한 작업 세션. 형식: 무엇을 / 왜 / 영향. 커밋 해시는 근거.

---

## 2026-06-25 23:35 — sync-arch 토큰 최적화 (이력 아카이브 분리)

**무엇을**
- ARCHITECTURE.md 변경이력(1줄 13,958자·27건)을 `ARCHITECTURE_CHANGELOG.md`로 분리 → 본문엔 `마지막 업데이트`+`최근 변경` 3건만(line7 248자·파일 65.8K→50.3K).
- WORKLOG.md 43블록 → 최근 5개만 유지, 38블록 `WORKLOG_ARCHIVE.md`로 이동(47.6K→5.2K).
- `/sync-arch` 스킬 갱신: 토큰 절약 원칙(아카이브 비독·head/limit·Grep)·CHANGELOG 분리 절차·WORKLOG 로테이션 스크립트(UTF-8 no BOM·`Get-Content` ANSI 함정 주석).

**왜**
- 사장: sync마다 토큰 과다 — 데이터 누적 탓. "최소화/최적화 또는 데이터 따로 저장". ARCHITECTURE.md는 작업 전 필수 읽기라 비대한 이력 1줄이 전 작업에 과금됨.

**영향 / 후속**
- 작업 전 ARCHITECTURE.md 읽기·sync 토큰 대폭 감소. 데이터 손실 0(git·아카이브 보존). 🔒 FROZEN·코드 미접촉.
- 앞으로 sync는 아카이브를 읽지 않고 최근 항목만 갱신.

**커밋**: `landing` (이 sync 문서)

---

## 2026-06-25 18:20 — 아침 시장메일 동적화 + 랜딩↔어드민 상품가격 동기화

**무엇을**
- **아침 시장조사 메일 동적화**(`15110e9`·`22a5b82`): `diagnoseMarketPhase(date)`가 월별 한국 채용시장 국면(1~12월 캘린더)을 진단 → **검색 그리드가 시기별로 유동 변화**(매일 비슷하던 문제 해소) + 전날 리포트 전달(미중복) + finance-data 사업현황 믹스(인사이트). 새 구조 📌인사이트→🗓️국면→🔍시기맞춤→🧭조직액션·제목에 국면. `cleanHtml`로 웹검색 머리말·코드펜스(```html) 제거. prod 검증 200/30.4s·제목'하반기 계획·인턴십 시즌'·4마커·출처4.
- **랜딩↔어드민 상품가격 동적 동기화**(`ef9dabd`): 공개 `GET /api/showcase/products`(`is_active&&is_public`) + 랜딩 카드 `data-product-key` 5종 + 로드시 fetch→가격 치환(실패 시 정적). 자소서 80→88·메이크업 160→170·연봉 40→50 자동교정. **어드민 가격만 바꾸면 랜딩 반영**(단일원천). prod 검증 endpoint 6상품·키5·스크립트 deploy.

**왜**
- 사장: ① "메일이 매일 똑같다 — 시기를 진단해 그 시기에 필요한 현황이 갱신되는 구조로". ② "어드민 상품과 랜딩 상품이 매칭 안 됨" → 동적 자동동기화(사장 선택).

**영향 / 후속**
- tsc 0·prod 스모크 통과. 🔒 FROZEN 미접촉(신규 라우트·랜딩 흡수원판만).
- ⬜ 사장(어드민 상품): 면접 가격 입력(현재 null→랜딩 정적)·랜딩 '서류+면접 자소서'(대응 상품 없음)·어드민 `package`(랜딩 카드 없음) 정리하면 매칭 완결.
- (참고) 가격은 페이지 로드시 client fetch로 갱신(짧은 깜빡임) — 무깜빡임·SEO는 2단계 React 변환 시 서버렌더.

**커밋**: `admin-tool` `15110e9`·`22a5b82`(시장메일)·`ef9dabd`(상품동기화) / `landing` (이 sync 문서)

---

## 2026-06-25 15:10 — 랜딩+admin-tool 코드병합 1단계 + 오더리포트 JSON 복구

**무엇을**
- **랜딩 흡수(코드병합 1단계)**(`97f2772`): 정적 랜딩(루트 `index.html`+`style.css`)을 admin-tool `public/landing.html`·`style.css`로 흡수. `next.config` beforeFiles rewrite `/`→`/landing.html`(다른 경로 불변). 랜딩의 `admin-tool-cyan` 절대URL→상대경로(`/showcase`·`/api/submit-diagnosis`: 같은 origin·CORS 제거). **prod 검증**: `/`=랜딩(200)·`/showcase` 200·`/admin` 307→login·`/style.css` 200.
- **오더 결과리포트 JSON 잘림 복구**(`ea52c3f`): 복합 오더에서 분신 AI JSON이 max_tokens에서 잘려 'Unterminated string' 파싱실패 → `repairTruncatedJson` 폴백 추가(`org-orders`·`org-execute` 공통)+토큰 2000→2600·timeout 48s·retry 0. prod 검증: 8액션 리포트 정상 생성.

**왜**
- 사장: 두 Vercel 프로젝트(정적 랜딩+Next admin-tool) 통합 결정 → 분석 후 '코드병합>프록시·1단계 안전 실행' 추천대로 진행. / 어드민 오더 리포트 생성 오류 신고.

**영향 / 후속**
- ⬜ 사장: 도메인 `rocketcareer.co.kr`을 admin-tool 프로젝트로 이전(Vercel Domains·무중단) → 단일 사이트 완성. 이전 전엔 `admin-tool-cyan.vercel.app/`에서 랜딩 프리뷰.
- (선택, 2단계) 랜딩 React/Tailwind 정식 변환. 랜딩 repo 원본 무변경(흡수만). tsc 0·build 0·🔒 FROZEN 미접촉.

**커밋**: `admin-tool` `ea52c3f`(JSON복구)·`97f2772`(랜딩병합)

---

## 2026-06-24 23:42 — 멘토 온보딩 규제 방어 5단계 (용역계약/윤리서약 분리)

**무엇을**
- 멘토 온보딩을 4→**5단계**로 재설계: `legal`(용역계약: 독립 프리랜서·업무범위 한정·성과 비연동·3.3% 세무) → **`pledge` 신설**(윤리서약: 알선금지·합격보장금지·비밀유지) → edu1(방향·역할+표준 업무범위=권장 안내서) → edu2(응대) → edu3(실무+정산·세무).
- `pledge` 알선금지·성과비연동 **개별 체크박스 2개**(둘 다 체크해야 활성, 서버 검증)·확인시각 기록(`brokerage_ack_at`·`outcome_pay_ack_at`). 동의 페이지 단계 **전문 표시**.
- 콘텐츠: 필수문언 6종, 금지표현(근무·급여·지시·전속·성공보수) 배제, '지시→범위·통제→합의·고용→용역' 프레이밍.
- **fix(auth)**: `verifyOnboardingToken` 단계 화이트리스트에 `pledge` 누락 → 토큰 거부 버그(②서약 링크 전면 차단) 수정. 화면 캡쳐 검증 과정에서 발견.

**왜**
- 사장 요구: 멘토 통제 인상 없이 자발적 규제 경계 준수(직업안정법 알선·노동법 근로자성 2선 동시 방어)를 계약 문언으로.

**영향 / 후속**
- tsc 0·build 0·5단계 렌더+게이트 검증·main FF 배포(`2395d9c`). 🔒 FROZEN 미접촉(신규 단계·컬럼·페이지만).
- ⬜ 사장: `supabase-mentor-onboarding.sql`(pledge·ack 컬럼) 적용(기록·게이트 활성)·**노무사/세무사 1회 검토**(초안)·정산 방식(고정액/%) 확정 시 ⑤ 대가 문구 반영.

**커밋**: `admin-tool` `2395d9c` / `landing` (이 sync 문서)

---

## 2026-06-23 22:36 — 오더 루프 라이브 검증·배포 + 후속 보완

**무엇을**
- **배포**: `supabase-org-orders.sql` 적용(사장) 후 `admin-tool` main FF 푸시 — `05d6e02`(루프)·`b61a76a`(parseJson)·`539a426`(보완). Vercel 프로덕션.
- **라이브 스모크 통과**(실제 prod 테이블): 등록→고위험차단→재분석 스레드→본부 배정→자율 실행+교차검수(sales→검수 bizdev) 한 바퀴, 테스트행 자동 삭제.
- **버그픽스**: `parseJson`이 JSON 뒤 산문에 깨져 재분석 보고서가 비던 문제 → 중괄호 매칭으로 첫 객체만 파싱(`b61a76a`).
- **후속 보완**(`539a426`): ① isHighRisk '배포/인증' 오탐 정제(명백키워드+구체문맥 분리, 검증 7/7). ② 자율실행 한도 60s→300s·timeout/토큰 상향(무거운 오더 수용). ③ `/admin/orders` 오더 카드 기본 접힘(collapsible).

**왜**
- 사장 "전부 해" → SQL 적용·배포·라이브 검증. 스모크가 parseJson 버그 발견. 사장 후속요청: 오탐 다듬기·카드 접기·한도 확대.

**영향 / 후속**
- tsc 0·build 0·스모크 PASS. 🔒 FROZEN 미접촉.
- ⬜ 진짜 무인 멀티스텝(heavy) = 워커 호스트(3b, Railway/Render) 사장 인프라 결정 대기. root 문서 푸시는 가드레일로 보류 가능.

**커밋**: `admin-tool` `05d6e02`·`b61a76a`·`539a426` (main 배포) / `landing` (이 문서·org)
