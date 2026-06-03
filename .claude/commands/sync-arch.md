# /sync-arch — 아키텍처 문서 동기화 + 작업 로그 기록

작업 완료 후 ① ARCHITECTURE.md와 실제 코드 상태가 맞는지 검증·업데이트하고,
② **이번 세션에 무슨 작업을 왜 했는지**를 일자·타임스탬프와 함께 `WORKLOG.md`에 누적 기록한다.

> 목적: 문서의 "현재 상태"만으로는 *어떤 과정을 거쳐 지금에 이르렀는지* 추적이 안 돼 혼선이 생긴다.
> sync 할 때마다 약식 작업 로그를 남겨 시간순 히스토리를 항상 트래킹한다.

## 실행 절차

1. `ARCHITECTURE.md` 읽기
2. 각 모듈 경로 존재 여부 확인:
   - `admin-tool/app/api/content/` — 마케팅 자동화
   - `admin-tool/app/api/local-studio/` — 로컬 영상 제작 패널 (dev 전용)
   - `admin-tool/app/api/submit-diagnosis/` — 구간3 (FROZEN)
   - `admin-tool/lib/marketing/` — 영상 파이프라인·TTS·스코어링 등
   - `admin-tool/remotion/` — Remotion 컴포지션
   - `Mento_incoding_casedb_project/henry-casedb/` — 인코딩 DB
   - `marketing_division/` — 마케팅 IMC 기획안 (전략 SoT, v1·v2)
3. 실제 파일 구조와 ARCHITECTURE.md 불일치 항목 리포트
3-A. **마케팅 전략(IMC) 정합 체크 (필수)** — 아래 "마케팅 정합 절차" 수행
4. 사용자 확인 후 ARCHITECTURE.md 업데이트
5. 변경된 모듈이 있으면 해당 모듈 CLAUDE.md의 `현재 상태` 섹션도 업데이트
6. **작업 로그 기록 (필수)** — 아래 "작업 로그 절차" 수행

## 마케팅 정합 절차 (IMC ↔ 코드)

> 코드(`lib/marketing` 등)만 검증하면 그 **상위 마케팅 전략(IMC)이 코드와 따로 노는 표류**를 못 잡는다.
> ARCHITECTURE.md "📣 마케팅 전략 레이어" 표를 기준으로 IMC 기획안 v2 ↔ 코드·자산 일치를 점검한다.
> (마케팅은 아키텍처의 일부다 — 코드 모듈과 동급으로 동기화한다.)

점검 항목 (기획안 § ↔ 코드·자산):
- BI 컬러: §1 ↔ `style.css` / `remotion/lib/fonts.ts` (골드크림 폐기 유지?)
- 브랜드 인트로: §1 ↔ `CaseFileVideo.tsx` (`INTRO_SEC`·연출·태그라인 위치)
- 캐치·발사 메타포: §3 ↔ 인트로 태그라인 / `generatePublishKit`
- 화법 룰·훅·CTA: §3 ↔ `casefile-script.ts` SYSTEM_PROMPT (+ 룰 **4곳 동기화** 여부: 코드·`casefile-qa.md`·`remotion/CLAUDE.md`·`casefile-script.md`)
- 콘텐츠 카테고리·믹스: §5 ↔ `content-scorer.ts` / `script-generator.ts`
- 퍼널·페르소나: §7·§2 ↔ 구간3·랜딩 (전략이 바뀐 경우만)

처리:
- 불일치 발견 → 어느 쪽이 최신인지 사용자 확인 → 기획안 또는 코드 갱신 → 양쪽 정합.
- 전략 결정이 신규 추가/변경됐으면 ARCHITECTURE.md "📣 마케팅 전략 레이어" 표도 갱신.
- 결과를 출력 형식 "📣 마케팅 정합(IMC)" 라인에 기재.

## 작업 로그 절차 (WORKLOG.md)

1. 현재 타임스탬프 획득: `date '+%Y-%m-%d %H:%M'` (KST 기준)
2. 이번 세션 커밋 참조(근거 자동 수집):
   - `cd admin-tool && git log --since="<오늘 00:00>" --pretty="%h %ad %s" --date=format:'%H:%M'`
   - 루트(랜딩) 저장소도 동일하게 (`admin-tool/`은 자체 git repo, 루트는 `rocketcareer-landing`)
3. `WORKLOG.md`(프로젝트 루트)에 **최신 항목을 맨 위에 prepend**. 없으면 새로 생성(아래 헤더 포함).
4. 한 세션 = 한 블록. 약식이라도 **무엇을 / 왜 / 영향·후속**이 보이게. 커밋 해시를 근거로 첨부.

### WORKLOG.md 형식
```
# 작업 로그 (WORKLOG)

> /sync-arch 실행 시 자동 누적. 최신 항목이 맨 위.
> 각 블록 = 한 작업 세션. 형식: 무엇을 / 왜 / 영향. 커밋 해시는 근거.

---

## YYYY-MM-DD HH:MM — <한 줄 제목>

**무엇을**
- 핵심 변경 1
- 핵심 변경 2

**왜**
- 배경·의사결정 한두 줄

**영향 / 후속**
- 의존 모듈·주의사항·다음 할 일

**커밋**: `admin-tool` <해시…> / `landing` <해시…>

---
```

### 기록 규칙
- 사실만. 추측·미완료를 "완료"로 적지 않는다. 미완은 "후속"에 남긴다.
- 커밋 안 한 변경도 로그엔 남기되, "커밋: (미커밋)"으로 명시.
- 기존 항목은 절대 수정·삭제하지 않는다(append-only 히스토리). 오타 정정만 허용.
- 길게 쓰지 않는다. 블록당 8~12줄 이내 약식.

## 출력 형식

```
[sync-arch 결과]
✅ 일치: 마케팅 자동화, 구간3, 인코딩 DB
⚠️  불일치: [모듈명] — [사유]
🆕 신규 감지: [경로] — ARCHITECTURE.md에 없음
🗑️  경로 없음: [모듈명] — 삭제됐거나 이동된 것으로 보임

📣 마케팅 정합(IMC): ✅ 일치 / ⚠️ 표류 [항목] — [기획안 §N vs 코드]
📝 WORKLOG 기록: YYYY-MM-DD HH:MM — <제목> (커밋 N건 근거)
마지막 업데이트: [날짜]
```
