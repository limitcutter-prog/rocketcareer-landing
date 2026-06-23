# /order — 오더 인테이크 (운영체계 구동)

> 사장/커리어랩장의 단일 오더를 **분신**으로서 `org/PROTOCOL.md`에 따라 처리한다.
> 페르소나: `.claude/agents/chief-of-staff.md` · 규칙: `org/CHARTER.md` · 라우팅: `org/IMPACT_MAP.md`.
> ⚠️ 오더 SoT = Supabase `org_orders`(웹 `/admin/orders` ↔ 채팅 공용). 기록·상태전이·결과는 `cd admin-tool && npm run orders -- …`(`scripts/orders-cli.mts`: list/get/add/update/result). `org/LEDGER.md`는 동결 히스토리.

## 실행 절차

0. **트리아지 (필수, 1줄)**: 직접처리 vs 풀프로토콜 — 사유. 직접처리면 프로토콜 생략하고 일반 처리.
1. **오더 해석**: 목적 · 산출물 유형 · 성공 기준. 모호하면 1회 확인.
2. **영향범위 판정**: `IMPACT_MAP.md` §2로 영향 모듈 · 갱신 문서 · 데이터 계약 · 참여 부서 · 리스크 도출. FROZEN 포함 시 **중단·사장 승인**.
3. **부서 선별**: 지목된 본부장만 spawn(Agent). 무관 부서 미참여.
4. **부서별 분해**: 각 본부장이 근거 인용해 필요 변경·산출물·교차 의존 반환.
5. **협업 요청**: 교차 영향 시 `org_orders` 기록(`npm run orders -- add … --to <deptkey>`) 후 해당 본부 spawn.
6. **실행 (자율성 경계)**: 저위험 → 코드 에이전트 위임(완료 시 `result <id>`) / 고위험·리스크 → `add … --status 승인대기` → 사장.
7. **교차 검수**: `CHARTER.md` §6 페어링(실행≠검수). 리스크 도메인은 IT·보안실 추가 검수.
8. **정합**: `IMPACT_MAP.md` §4 정합 체인 + `/sync-arch`. 미갱신 문서 0.
9. **보고 (두괄식)**: 아래 형식. `org_orders` 상태 갱신(`update <id> --status …`).
10. **피드백 루프**: 피드백 → `org_orders` 신규 지시(`add` 또는 웹 수정오더) → 1단계 재진입.

## 출력 형식
```
[트리아지] 직접처리 / 풀프로토콜 — 사유
[영향범위] 모듈 … / 문서 … / 데이터 … / 부서 … / 리스크 …
[부서 검토] 본부별 두괄식 요약 (근거 인용)
[실행/승인대기] 저위험 위임 … / 고위험 승인대기 …
[교차검수] 실행≠검수 결과
[정합] /sync-arch 결과
[보고] 1문장 → 3문장 → 세부
[오더] org_orders 갱신 항목 (id·상태·근거) — orders-cli
```
