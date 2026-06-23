# /standup — 일일 운영 점검 (분신)

> 페르소나: `.claude/agents/chief-of-staff.md`. `OBJECTIVES.md` 기준으로 본부 현황을 모아
> 두괄식 보드 리포트를 내고 **오더(`org_orders`)** 를 추진한다. 인자 `full`이면 6개 본부 전부 점검.
> ⚠️ 오더 SoT = Supabase `org_orders`(웹·채팅 공용). 조회·기록은 `npm run orders -- …`(`admin-tool/`). `org/LEDGER.md`는 동결 히스토리(읽기 참고만).

## 실행 절차

1. `org/OBJECTIVES.md` + `org/state/*.md` + **오더 목록**(`cd admin-tool && npm run orders -- list`) 읽기. (히스토리 맥락은 `org/LEDGER.md` 참고)
2. **본부장 spawn**: 기본은 열린(`승인대기`/`지시`/`진행`) 오더·분기 우선순위에 걸린 본부만(focused, 비용 절약). `full`이면 6개 전부.
   - 각 본부장은 현 OBJECTIVES 기준 **현황·목표정합·블로커·다음제안**을 근거 인용해 반환.
3. **종합**: 현 목표 기준 교차본부 **최우선순위 재도출**(목표가 바뀌면 우선순위도 바뀜 = 유동성).
4. **상태 갱신**: 각 `org/state/*.md`의 "현재 목표 정합"·"다음 사이클 제안" 갱신.
5. **오더 기록**: 신규 지시는 `npm run orders -- add --content "…" [--to <deptkey>] [--report '<json>']`. 진행/완료는 `update <id> --status …`, 실행결과는 `result <id> --summary …`. 저위험 자동위임 / 고위험 `승인대기`.
6. **보고**: 두괄식.

## 출력 형식
```
[결론] 1문장
[요약] 3문장
[본부별 현황] 본부 — 한 줄 현황(근거) · 블로커
[교차 최우선 3] 1. … 2. … 3. … (OBJECTIVES 연결)
[LEDGER 갱신] ID·수신·상태
```

> 객관성: 근거 없는 "양호"·"완료" 금지(CHARTER §1). 데이터 없으면 "측정 불가"로 명시.
