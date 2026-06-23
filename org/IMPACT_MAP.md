# 영향범위 라우팅 테이블 (IMPACT MAP)

> `PROTOCOL.md` 2단계 "영향범위 판정"이 이 표를 **라우팅 테이블**로 사용한다.
> 원천: `ARCHITECTURE.md`(의존성 맵·변경 임팩트 체크표·📣 마케팅 전략 레이어) 결정화.
> **ARCHITECTURE.md가 갱신되면 이 표도 동기화한다** (`/sync-arch` 점검 대상).

---

## 1. 모듈 ↔ 부서(소유) 매핑

| 모듈 | 소유 본부 | 협업/검수 | 실행 에이전트 |
|---|---|---|---|
| 1 구간3 (🔒FROZEN) | IT·보안실 (수호) | (수정금지) | — |
| 2 마케팅 자동화 | 마케팅본부 | R&D(대본 논리) | marketing-db-agent |
| 3 어드민 UI | IT·보안실 | 해당 기능 본부 | admin-upload-agent |
| 4 대시보드(유입분석) | 마케팅본부 | 경영지원(지표 해석) | admin-upload-agent |
| 5 트렌드/Bitly | 마케팅본부 | — | marketing-db-agent |
| 6 멘토 관리 | 영업본부 | IT·보안실 | admin-upload-agent |
| 7 영상 자동생성 | 마케팅본부 | IT(렌더 정상) | — |
| 8 인코딩 DB | R&D본부 | 마케팅(케이스 공급) | encoding-agent |
| 9 랜딩페이지 | 마케팅본부 | IT·보안실 | landing-agent |
| 10 Auth/포털 | IT·보안실 | 영업(멘토포털) | — |
| 11 리포트/PDF | R&D본부 | IT(서버리스 PDF) | admin-upload-agent |
| 12 마켓플레이스 | 영업본부 | 사업개발(상품)·경영지원(수익)·IT | — |
| 13 멘토링 운영 | R&D본부 | IT(STT/PDF)·경영지원(상품) | — |
| 14 AI 운영체계(오더) | 분신 | IT·보안실(코드)·커리어랩장(목표) | 6 본부장 |
| 📣 IMC 전략 | 마케팅본부 | R&D(화법 논리) | — |

## 2. 변경 트리거 → 영향범위 라우팅

> `ARCHITECTURE.md` 변경 임팩트 체크표 + 문서 정합 + 부서 + 리스크 통합.

| 변경 대상 | 영향 모듈 | 갱신 문서 (정합) | 데이터 계약 | 참여 부서 (실행/검수) | 리스크 |
|---|---|---|---|---|---|
| 콘텐츠 화법·훅·CTA 룰 | 2, 7 | IMC v2 + **4곳**(`casefile-script.ts`·`casefile-qa.md`·`remotion/CLAUDE.md`·`casefile-script.md`) | — | 마케팅 / R&D | — |
| `cases_career` 컬럼 | 2, 8 | ARCHITECTURE 모듈8 | `content-scorer.ts` 직접참조 · **JSONB 함정** | R&D / 마케팅 | PII |
| `content_queue` 컬럼 | 2, 3, 7 | ARCHITECTURE 모듈2 | `content/review` TS 타입 · `casefile_status` 잡큐 | 마케팅 / IT | — |
| 진단 파이프라인 | 1 🔒 | — | mentees/diagnoses | (중단·사장 승인) | **FROZEN** |
| 멘토링 회차·리포트 | 13 | `PLAN.md` · SOP v1.1 | `case_*` 신규(FROZEN 읽기 FK만) | R&D / IT | PII·녹취·STT·리포트 |
| 멘토·거래·결제 | 12 | PRD · IMC §1 | matches/orders/settlements | 영업 / 사업개발·경영지원 | 결제·정산·인증 |
| 가격 티어 | 사업, 12 | `00_사업전체_참고문서` · IMC | `services.price_band` | 사업개발 / 경영지원 | **가격정책(고위험)** |
| 인증·미들웨어 | 3, 10, 12 | ARCHITECTURE 인증경계 | `middleware.ts` PUBLIC_API | IT·보안실 / — | **인증(고위험)** |
| 상품·시장 전략 | 사업, 전체 | `OBJECTIVES.md` · IMC | — | 사업개발 / 커리어랩장 | — |
| 조직 R/R | (운영체계) | `ORG_CHART.md` | — | 경영지원 / 분신 | — |
| 세무·정산 | (운영체계)·12·13 | `ORG_CHART.md` · finance-data | 원천징수·증빙·정산 | 경영지원(`tax-advisor`) / 사업개발 | 세무(자문한계·세무사확인) |

## 3. 데이터 계약 (변경 시 다운스트림 즉시 통보)

- `cases_career.*` → `content-scorer.ts`·`script-generator.ts`(모듈2). 🔴 **JSONB 함정**: `improvements`·`weaknesses_before`·`weaknesses_remaining`는 Supabase가 **string으로 반환** → `JSON.parse` 필수.
- `content_queue.*` → `content/review` TS 타입(모듈3), `casefile_status` 잡큐 계약(모듈7).
- `trend_benchmarks` 9항목 → `script-generator` 벤치마크 주입.
- 🔒 FROZEN 테이블(`mentees`·`diagnoses`·`contracts`·`session_notes`·`mentors`) → **읽기 FK 조인만**.
- `case_*` 신규(모듈13) → `session_reports` 8섹션, `case_artifacts` JSONB.
- 🆕 `org_orders`·`org_order_events`(모듈14, 오더 SoT) → 웹 `/admin/orders` ↔ 채팅 `orders-cli.mts` 공용. `report`/`result` JSONB. `org/LEDGER.md`는 동결 히스토리. 마이그레이션 `supabase-org-orders.sql`.

## 4. 문서 정합 체인 (코드 ↔ 문서 강제)

- IMC 화법 룰 변경 → **4곳 동시 갱신** + `/sync-arch`.
- 코드 모듈 변경 → `ARCHITECTURE.md` 해당 섹션 + `/sync-arch` + `WORKLOG.md`.
- 멘토링 운영 변경 → SOP v1.1 + `PLAN.md`.
- 전략 변경 → IMC v2 + `OBJECTIVES.md`.
- 어떤 변경이든 문서 미갱신 시 `/sync-arch`가 "표류"로 리포트한다.

## 5. 🔒 FROZEN 경계 (절대 수정 금지 — 발견 즉시 중단·사장 승인)

경로: `submit-diagnosis/` · `inbox/` · `send-email/` · `lib/anthropic.ts`(export) · `lib/supabase.ts`(export).
테이블: `mentees` · `diagnoses` · `contracts` · `session_notes` · `mentors`.
우회 원칙: 신규 테이블(`case_*` · `mentor_*` 등)로 분리, FROZEN은 FK 읽기조인만.

## 6. 리스크 플래그 인덱스

위 §2 "리스크" 열에 표기된 항목 → `CHARTER.md` §5 리스크 검수 + §4 자율성 경계 적용.
PII · 녹취 · STT · 리포트/PDF · 인증 · 결제 · 정산 · FROZEN · 가격정책.
