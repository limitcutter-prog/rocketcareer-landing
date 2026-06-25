> **🌐 랜딩페이지 SoT (2026-06-25 통합)**: 랜딩 수정·작업은 **`admin-tool/public/landing.html`(+`admin-tool/public/style.css`)** 에서만 한다. 통합 사이트는 **admin-tool 하나**가 `/`(랜딩)·`/showcase`(매칭)·`/admin`(어드민)을 모두 서빙(`admin-tool/next.config.ts`의 beforeFiles rewrite `/`→`/landing.html`). 루트 `index.html`/`style.css`는 **레거시**(도메인 `rocketcareer.co.kr` 이전 후 폐기 예정 — 수정 금지). 새 랜딩 요청은 통합 원판에서 진행.

## 운영체계 — 오더 인테이크 (최우선)

> 회사는 AI 자율 협업 운영체계로 돌아간다. **모든 오더는 먼저 트리아지를 거친다.**
> 절차: `org/PROTOCOL.md` · 규칙: `org/CHARTER.md` · 라우팅: `org/IMPACT_MAP.md`

### 하이브리드 트리아지 (모든 오더 1줄 분류)
- **직접 처리**: 단순 질문·단일 파일·사소한 수정·단일 부서 명백 작업 → 프로토콜 생략.
- **풀 프로토콜 (`/order`)**: 2개+ 부서/모듈 영향 · 데이터/문서 정합 변경 · 리스크 도메인(CHARTER §5)·고위험(§4) · 판단형("전략/개선/왜 안 되지") → `/order`로 처리.

### 조직 (`org/`)
- 계층1: 커리어랩장(`OBJECTIVES.md`) · 분신(`LEDGER.md`)
- 계층2: 6본부 (`.claude/agents/*-head.md`, 상태 `org/state/*.md`)
- 계층3: 실행 에이전트 6종 (아래 표)
- 주기: `/standup`(일) · `/strategy-review`(주) · `/org-audit`(객관성 검증)

### 불변 원칙 (CHARTER 요약)
- 근거 없는 주장 금지(증거 등급: 조회수=참고, S5 결과=확정). 두괄식 보고. 피드백은 LEDGER 검증까지. 실행≠검수.
- 저위험 자동 실행 / 고위험(FROZEN·결제·인증·DB migration·배포·가격) 사장 승인.

---

## 거버넌스 워크플로우 (필수)

### 모든 작업의 필수 순서
1. **작업 시작 전**: `ARCHITECTURE.md` 읽기 → 변경 대상 모듈 확인 → 의존성 영향 파악
2. **모듈 작업 시**: 해당 모듈의 `CLAUDE.md` 읽기 (없으면 없는 대로 진행)
3. **🔒 FROZEN 확인**: `구간3 파이프라인` 모듈은 절대 수정 금지. 해당 경로 발견 즉시 작업 중단 후 확인
4. **작업 완료 후**: 변경된 모듈의 `CLAUDE.md` 업데이트 + `ARCHITECTURE.md` 상태 반영

### 변경 임팩트 판단 기준
- 스키마(DB 컬럼) 변경 → 의존 모듈 담당 에이전트에 즉시 전달
- API 인터페이스 변경 → `ARCHITECTURE.md` 데이터 계약 섹션 업데이트 필수
- 새 모듈 추가 → `ARCHITECTURE.md` 인벤토리에 추가 후 작업 시작

---

## 에이전트 오케스트레이션 규칙

### 에이전트 역할 요약
| 에이전트 | 담당 모듈 |
|---------|----------|
| landing-agent | 랜딩페이지 제작 |
| mentee-form-agent | 멘티 입력 FORM + PDF |
| resume-word-agent | 경력기술서 Word 생성기 |
| encoding-agent | 멘토 인코딩 작업 |
| admin-upload-agent | 어드민 업로드 |
| marketing-db-agent | 마케팅-어드민 DB 연계 |

### 병렬 처리 (동시 투입 가능한 조합)
- landing-agent + admin-upload-agent (UI 작업 독립적)
- mentee-form-agent + resume-word-agent (입력→출력 독립적)
- encoding-agent + marketing-db-agent (데이터 작업 독립적)

### 순서 처리 (의존성 있는 조합)
- DB 스키마 변경: marketing-db-agent → encoding-agent → admin-upload-agent 순
- 신규 폼 필드 추가: mentee-form-agent → resume-word-agent 순

### 에이전트 간 소통 파일
- PLAN.md: 스키마 변경, API 스펙, 의존성 명시 공유
- `org/LEDGER.md`: 부서 간 지시·협업요청·피드백 추적 (운영체계)
- `org/state/*.md`: 본부별 현황·블로커·목표 정합
- `org/IMPACT_MAP.md`: 변경 영향범위 라우팅 (부서 선별)

> 상위 경영 레이어(커리어랩장·분신·6본부)는 `org/ORG_CHART.md` 참조. 위 6개 코드 에이전트는 계층3 실행 단위(IT·보안실 산하).
