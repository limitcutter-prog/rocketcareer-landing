# 작업 로그 (WORKLOG)

> /sync-arch 실행 시 자동 누적. 최신 항목이 맨 위. **최근 5개 블록만 유지**(초과분 → `WORKLOG_ARCHIVE.md`).
> 각 블록 = 한 작업 세션. 형식: 무엇을 / 왜 / 영향. 커밋 해시는 근거.

---

## 2026-07-01 20:42 — 멘토링 S0~S4 멘티 입력·AI 반영 개선 10건 (모듈13)

**무엇을**
- 메일 2종 '7일 유효' 문구 제거 + 초대 메일 5단계 네비·가이드(인사담당자 톤). 포털 '시작 전 안내' 인사담당자 톤 재작성 + 진행단계 안내.
- 멘티 포털 넓은 2단 레이아웃(max-w-5xl). S1: 끌린 이유 4분리(당장/배움/5년/10년) + 채용공고·마감 분리 + 빨간 작성가이드 + 최악상황 문구. S2: '무엇에 기여했나' 카테고리+설명 다중 추가 + 작성가이드.
- S3 문서유형 분기: 경력기술서=회사>대>중>소 무한추가 트리 / 자소서=가이드+초안. 멘토가 S2에서 `case_journeys.doc_type` 지정(미설정 시 신입=자소서·경력=경력기술서).
- **병목 `serializeSubmission`에 전 필드 반영** → group-suggest·experience-suggest·report-generator 누락 없이 전달. SubmissionView(멘토 뷰)·AI 프롬프트 2종 동기화.

**왜**
- 사장: 멘티가 회차 입력에서 정보를 충분히 남기고, 그게 AI 요약(그룹화·합리화·리포트)에 빠짐없이 반영되게. 안내 문구는 클로드스럽지 않게 인사담당자 톤으로.

**영향 / 후속**
- tsc 0·next build 0·main FF 배포(`edbeb45`). 🔒 FROZEN 미접촉·case_* 신규 컬럼만.
- ⬜ 사장: `supabase-mentoring-doctype.sql` 1줄 적용 시 멘토 명시 문서유형 지정 활성(미적용 시 트랙 기본값으로 정상). 프리뷰는 인증 게이트라 배포 후 테스트 멘티로 스모크 권장.

**커밋**: `admin-tool` `edbeb45` / `landing` (이 sync 문서)

---

## 2026-06-26 18:41 — 멘토 셀프 어필 지면 + 매칭검토(사장 승인)

**무엇을**
- **멘토 지면(Pillar A)**: `mentor_blocks`(하이브리드 블록 9종) + `mentor_profiles`(cover/tagline/theme/accent) + 공개 버킷 `mentor-assets`. API `mentor/blocks`(+[id])·`mentor/upload`·`mentor/me` 확장·공개 `showcase/mentors/[id]`. 신규 풀페이지 `showcase/mentors/[id]` + `MentorBlocks.tsx` + 메인 `MentorSpotlight`(카드→풀페이지·모달 폐지) + 멘토 포털 지면 에디터(업로드·순서·미리보기).
- **매칭검토(Pillar B)**: 신청 status='review' → `/admin/matches`+`/api/matches`(+[id], OWNER_ONLY) 승인/반려 → matches/match_events/메일(멘티·멘토). 멘토 직접 수락/거절 폐지(requests PATCH→403, 포털 확인만). 권한·네비 추가.

**왜**
- 사장: 멘토가 이미지·글·별도형식으로 스스로를 어필하는 감각적 지면 필요(카드 나열 탈피) + 멘티 신청을 사장이 직접 승인하는 매칭검토 구조.

**영향 / 후속**
- ✅ `supabase-mentor-showcase.sql` 사장이 SQL Editor 적용 완료(테이블·컬럼) + 버킷 생성됨. tsc 0·API 신규필드·라우트 가드(401/307/200)·풀페이지 DOM 렌더 검증.
- 라이브 블록작성·매칭승인은 멘토/사장 세션 필요(헤드리스 미검증) — 코드·가드·타입 커버. 결제/정산 범위 외.
- 🔒 FROZEN 무접촉(마켓플레이스 테이블만). ⚠️ 배포는 `main`(DDL 선적용됨).

**커밋**: `admin-tool` `40b0809` / `landing` (이 sync 문서)

---

## 2026-06-26 17:44 — 어드민 상품관리 ↔ 랜딩 상품카드 정합

**무엇을**
- 랜딩 동기화 JS(`public/landing.html`)를 가격만 → **제목·소개·가격 전체** 치환으로 확장(`closest('.service-card')`), 카드#3에 `data-product-key="package-doc"` 추가, 정적 폴백가 정합(makeup 170k·coverletter 88k·salary 50k).
- DB 정합(사장 승인·service-role): 5개 상품 title·intro를 랜딩 카피로, interview price null→109,000, sort_order 시각순, `package-doc` 신규(90,000), `package` is_public=false. `supabase-products-sync.sql`(멱등)+시드 `supabase-products.sql` 반영.
- 어드민 안내문 갱신(`products/page.tsx`): 사이트 실시간 반영·시작가 명시.

**왜**
- 어드민 상품관리가 메인 사이트 상품 카드와 매칭 안 됨(가격만 동기화·제목 드리프트·미매칭 카드/고아 상품). 어드민을 카드의 단일원천으로 정렬.

**영향 / 후속**
- 공개 상품 6 ↔ 랜딩 카드 6:6 1:1. 어드민 편집이 즉시 카드에 반영. preview 검증 통과(콘솔 에러 0).
- ⚠️ 랜딩 배포 시 DB 정합이 선행돼야 함(이미 라이브 적용됨). title 평문화로 수동 `<br/>` 줄바꿈은 CSS 자동 줄바꿈으로 대체(경미).
- 🔒 FROZEN 미접촉. 가격 외 변경 없음(interview만 null→표시값 109k).

**커밋**: `admin-tool` `a65f814` / `landing` (이 sync 문서)

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
