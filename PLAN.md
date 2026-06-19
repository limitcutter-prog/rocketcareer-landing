# PLAN.md — 에이전트 간 스키마·API·의존성 공유

> 거버넌스: 스키마 변경·API 스펙·의존성은 이 파일에 명시해 에이전트 간 공유한다 (CLAUDE.md).
> 상위 인덱스: `ARCHITECTURE.md` · 모듈 상세: 각 `CLAUDE.md`.

---

## 🟢 활성 계획 — 모듈 13: 멘토링 운영(딜리버리) 시스템

**상태:** 📋 PLANNED (0단계 거버넌스 등재 완료 2026-06-11 / 1~3단계 미착수)
**담당(예정):** admin-upload-agent(1단계 UI·API) + encoding-agent/마케팅엔진(2단계 STT·리포트 생성)
**SoT 문서:** `admin-tool/mentoring tool/files/멘토링_운영표준_SOP_v1.md` (SOP v1.1), `.../멘토_운영스크립트_v1.md` (v1.0)

### 1. 배경·근거
두 운영 문서는 **"결제 이후 5세션(S0~S4)을 실제로 돌리고, 회차마다 녹취→리포트를 만들고, 종료까지
케이스를 관리하는 멘토링 실행(딜리버리) 시스템"** 을 정의한다. 현재 어드민 툴은 ① 진단(FROZEN) ②
마케팅/영상 ③ 마켓플레이스 3축으로 발달했고, 이 **딜리버리 축은 `app/admin/mentees/[id]`에 씨앗
(계약·세션메모·파일)만** 있다. SOP가 "자동화 1순위"로 지정한 **녹취→STT→리포트 파이프라인은 코드 0건**.

### 2. 확정 결정 (2026-06-11, 대표)
1. **상품모델**: SOP의 "단일 OT+4회차=5세션" = 현재 `makeup` 상품(**"회사선택→이력서 Make-up"**, 200,000원).
   - 매핑: S0=OT(진단) / S1=타깃 그룹화(회사선택 Roadmap) / S2=경험 구체화 / S3=심화과정(기본형 문서·경력기술서·자소서) / S4=심화과정(추가)·종료.
   - 회차 운영은 **makeup 계약에만** 연결. 나머지 5종(`resume`·`coverletter`·`package`·`interview`·`salary`)은 단발 가치-업 서비스로 유지(회차 비대상).
   - 가격·패키지 재편 없음(매핑만 기록).
2. **FROZEN 우회**: 회차·리포트 데이터는 **신규 테이블로 분리**.
   - 모듈1 FROZEN(`mentees`·`contracts`·`session_notes`) **스키마 무수정**.
   - `mentees`·`contracts`는 **FK 읽기조인만** (`case_journeys.contract_id` → makeup 계약).

### 3. 신규 스키마 계약 (구현 시 조정 가능, 변경 시 이 표 갱신)

> 전부 신규 테이블. 멀티멘토 대비 `mentor_id` 선반영 권장. JSONB는 Supabase가 문자열 반환 가능 → `JSON.parse` 방어 필수(모듈8 교훈).

| 테이블 | 역할 | 핵심 컬럼 | 근거(SOP §) |
|---|---|---|---|
| `case_journeys` | makeup 계약 1건 = 멘토링 여정 1건 | `id, mentee_id(FK), contract_id(FK), track('newbie'\|'experienced'), status, ot_date, deadline_at(=ot_date+3개월), last_contact_at, urgent(bool), created_at` | §1 트랙분기 / §2.2 이용기간 3개월·30일 리마인드·휴면 |
| `case_sessions` | 회차 S0~S4 | `id, journey_id(FK), code('S0'~'S4'), status('locked'\|'assignment_pending'\|'scheduled'\|'done'), scheduled_at, duration_min, assignment_done(bool), recording_url, stt_text, judgment_note(재량 1줄), created_at` | §3 회차 SOP / §2.1 45분 / 과제 게이트 / §2.5 재량 기록 의무 |
| `session_reports` | 표준 8섹션 회차 리포트 | `id, session_id(FK), sections(JSONB 8섹션), stt_source, status('draft'\|'reviewed'\|'sent'), pdf_url, reviewed_at` | §5 리포트 제작 규정 |
| `case_artifacts` | 작업 산출물(그룹·경험·placeholder) | `id, journey_id(FK), target_groups(JSONB: 테마명·소속공고·어필관점·제외사유), appeal_tags(JSONB 자유태그), representative_experiences(JSONB), placeholders(JSONB: text·status 미결/해소)` | §3-S1 그룹화 / S2 어필태깅·합리화 / S3 placeholder 추적 |

**설계 원칙 (SOP §0·§6 반영):**
- 절차·산출물·기록은 **구조화**, 판단은 **재량 보존 + 근거 기록 의무**(규칙으로 대체 금지).
- **6축 점수는 멘토 비개입**(자기인식 장치) → **멘토 보정 점수 필드 두지 않음**(PDR 듀얼스코어 폐기).
- 어필포인트·KEY 태그는 **개방형 자유 태깅 + 사후 빈출 집계**(고정 enum 금지).
- 파일명 규격 §2.5: `{멘티ID}_{회차ID}_{문서유형}_{vN}` → 기존 `files`(intake/draft/final) 카테고리 위에 회차ID 부여.

### 4. 단계별 로드맵

| 단계 | 내용 | 위험 | 선행 블로커 |
|---|---|---|---|
| **0** ✅ | 거버넌스: PLAN.md + 모듈13 ARCHITECTURE 등재 (문서) | 없음 | — |
| **1** | 멘티 상세에 **회차(S0~S4) 탭** + 과제 게이트 + 재량 1줄 기록. `case_journeys`·`case_sessions` 신규. 기존 [mentees/[id] 상세 UI](admin-tool/app/admin/mentees/[id]/page.tsx) 재사용 | 저 | 신규 SQL 실행 |
| **2** (자동화 1순위) | 녹취 업로드 → **STT**(네이버 클로바/Whisper) → **8섹션 리포트 에이전트**(Claude) → 멘토 검수 → **PDF**. `session_reports`·`case_artifacts` | 중 | **`lib/pdf.ts` `throw` 복구**(서버리스 대안 결정) |
| **3** | 멘토 포털에 **운영 스크립트·S0 진단 체크리스트·메시지카드 9종·실수Top7** 임베드 + **예외처리**(긴급전형/휴면/리스크소재) 상태머신 | 중 | 모듈12 [멘토 포털](admin-tool/app/mentor/page.tsx) 연동 |

### 5. 의존성·영향
- **FROZEN 무수정 보장**: 신규 `case_*` 테이블만 생성. 모듈1 경로(`submit-diagnosis`·`inbox`·`send-email`)·스키마 무변경.
- **PDF 블로커**: [lib/pdf.ts:19](admin-tool/lib/pdf.ts:19) `generatePdf()`가 "로컬 환경에서만 지원" `throw`. 2단계 전 서버리스 대안(로컬 렌더 패턴 차용 / 외부 서비스 / 클라이언트 HTML→PDF) 결정 필요.
- **재사용 자산**: 멘티 상세 페이지(계약·메모·파일 그릇), `STAGE_TEMPLATES`([contracts/route.ts:4](admin-tool/app/api/mentees/[id]/contracts/route.ts:4)) — makeup의 단발형 stages는 회차 구조로 대체, 멘토 포털 UI.
- **현행 makeup `stages`**: `['자료 수령','초안 작성','피드백 수신','수정 완료','최종본 전달']`(단발 납품) → S0~S4 회차로 **대체**(stages JSONB는 모듈1 컬럼이므로 값만 변경, 스키마 무수정).

---

## 📒 처리 완료/보류 이력
- (2026-06-11) 모듈 13 0단계 등재.
- (2026-06-11) 모듈 13 **1단계 코드 완료**: `supabase-mentoring-ops-setup.sql`(case_journeys·case_sessions) + journey API(GET/POST 5회차 시드/PATCH) + sessions PATCH + `JourneyPanel.tsx`(멘티 상세 삽입). `tsc --noEmit` 에러 0.
- (2026-06-11) 모듈 13 **1단계 SQL 적용·E2E 검증 완료** (여정 생성·회차 PATCH·이용기간 OT+3개월). 피드백 반영: ① `makeup` 계약 시에만 노출 · ③ 회차 순차 잠금.
- (2026-06-11) 모듈 13 **AI 보조 레이어 첫 구현(피드백②)**: S1 그룹화 — `lib/mentoring/group-suggest.ts` + `suggest-groups` API + S1 UI, `case_sessions.ai_suggestion`·`assignment_input` 컬럼. 실 Claude 호출 E2E 검증(테마묶기·제외·직무교정 SOP 재현). 공고명 보존 6/10 → 후속 튜닝 과제.
- (2026-06-11) 모듈 13 **S2 경험 합리화 AI** + sync-arch(WORKLOG 기록): `experience-suggest.ts` + `suggest-experience`(**S1 그룹 자동참조**) + S2 UI. E2E 검증(번역·과대표현 정직화·종교 리스크·대표경험).
- (2026-06-11) 모듈 13 **S2 환각 해결**("S2 개선"): 입력 줄단위 ref 매핑 + `experience` 원문 강제 교체 → 창작/누락 0(6/6 원문 일치 검증).
- (2026-06-11) 모듈 13 **S2 파싱 잘림 수정**(실사용 에러): 긴 입력에서 출력이 `max_tokens`(4000) 잘려 JSON 파싱 실패 → 8000 + 출력 압축.
- (2026-06-11) 모듈 13 **S2 잘림 완전 해결**(8000도 넘는 긴 입력): ① `repairTruncatedJson`(잘려도 완료분 복구 → 502 방지) ② experience 모델 생성 생략(빈문자열, 코드가 ref로 원문 복원 → 토큰 절반·속도 2배). 20항목 E2E 48s·원문 100%·502 없음.
- (2026-06-13) 모듈 13 **2단계 — 멘티 전달 리포트**(사용자 요청): `report-generator`(AI 가공: 핵심·직무맥락·임팩트·세부·다음액션, SOP §5) + `report-pdf.tsx`(react-pdf, Noto Sans KR) + Resend 메일. `case_sessions`에 `mentee_report`·`report_status`·`report_sent_at`. UI: SessionCard 리포트 영역(생성·미리보기·PDF·메일). tsx 검증(리포트+PDF 52KB·한글). ⚠️ **SQL 재실행 필요**. dev OOM은 `dev.mjs` 8GB로 완화(프로덕션 무관).
- (2026-06-13) 리포트 톤 **현직 인사담당자 관점 강화**: report-generator SYSTEM_PROMPT — 멘토=대기업 HR 출신, 각 highlight에 "채용담당자가 서류·면접에서 어떻게 읽는지" 명시, 합니다체 공감형. 검증: "채용담당자는 8D 경험을 ~로 읽습니다" 톤 확연. 메일 from은 `onboarding@resend.dev`(미검증)라 본인 이메일만 — 도메인 검증 시 해소(코드 무관).
- (2026-06-13) **S1/S2 리포트 구조 재설계**: `S1GroupCard` + `S2ExperienceMapping` + `RoadmapInfo` 인터페이스 추가. S1=그룹별 카드(fit_summary·appeal_points·approach_tips·hr_read), S2=경험×그룹 포지셔닝 매트릭스, 전 회차=로드맵 5단계 인디케이터. report/route.ts에서 S2 생성 시 S1 ai_suggestion 자동 참조. report-pdf.tsx에 GroupCards·ExperienceMatrix·Roadmap 컴포넌트 추가. JourneyPanel.tsx UI 미리보기도 동기화. tsc 에러 0.
- (2026-06-18) 모듈 13 **멘티 포털(멘티 화면) 신규 — Phase A~C 코드완료·tsc 0**: 이메일 매직링크 인증(`lib/auth` 멘티 헬퍼·`middleware` 멘티 게이트) + `app/mentee/*`(login·dashboard·forms S0~S4) + `app/api/mentee/*`(me·sessions 제출·files·report) + `lib/mentee-portal`(소유권·노출 화이트리스트·직렬화). 멘티 제출 → `case_sessions.mentee_submission` + `assignment_input` 직렬화 → 멘토 JourneyPanel "멘티 초대"·제출 배지로 자동 표시(AI 제안 무수정). SQL `supabase-mentee-portal.sql`(case_sessions 2컬럼·files.session_id, **미적용**). 🔒 FROZEN 읽기조인만 + 전 라우트 소유권 검증. 후속(사장 승인): SQL 적용·라이브 스모크·매직링크 메일.
