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
