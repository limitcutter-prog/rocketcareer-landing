# 작업 로그 (WORKLOG)

> /sync-arch 실행 시 자동 누적. 최신 항목이 맨 위.
> 각 블록 = 한 작업 세션. 형식: 무엇을 / 왜 / 영향. 커밋 해시는 근거.

---

## 2026-05-30 01:16 — 로컬 스튜디오 제작 이력 영속화 + CTA 톤 개편(진단 유도)

**무엇을**
- 로컬 스튜디오 **제작 이력 영속화**: 렌더 시 `out/_studio-index.json`에 caseId별 기록(파일·케이스번호·훅·직무·시각). 신규 `GET /api/local-studio/renders`(mp4 잔존분만). 목록에 `✅ 제작됨` 배지+제작일시, 케이스 전환 후에도 기존 영상 미리보기 자동 복원(대본 생성 전에도 표시).
- **CTA 톤 개편**: "첨삭" 직접 권유 → **커리어 진단/점검·고민** 프레이밍 3톤(점검 권유/호기심 자가진단/공감 셀프체크) 중 케이스에 맞는 1개 변주. 짧게(한 줄 ≤18자·2~3줄). 저장+댓글 2요소·시리즈 예고 금지 유지. QA hard-fail에 "첨삭 직접권유·줄깨짐 길이" 추가.
- 룰 4곳 동기화: `casefile-script.ts`(룰8·JSON예시·QA) / `remotion/CLAUDE.md`(룰6) / `.claude/commands/casefile-script.md`·`casefile-qa.md`.

**왜**
- 다른 케이스 누르면 영상이 사라지고 제작 이력도 안 보여 혼선 → 이력 박제·복원.
- 마지막 후킹을 첨삭 직접 언급보다 커리어 전반 진단 유도가 전환에 유리(사용자 판단). 영상 여러 개 양산이라 톤 변주 + 자막 줄 안깨지게 짧게.

**영향 / 후속**
- `out/_studio-index.json`은 로컬 산출물(.gitignore 대상, 커밋 안 함). mp4 수동 삭제 시 `renders`가 자동 제외.
- CTA 톤은 다음 대본 생성부터 적용 — 실제 렌더로 줄바꿈·길이 체감 후 미세조정 권장.
- 검증: `tsc --noEmit` 통과, dev 서버에서 `/renders` 200·케이스 27 대본 생성 정상.

**커밋**: (미커밋) — `admin-tool` page.tsx·render/route·casefile-script.ts·remotion/CLAUDE.md + 신규 renders/route.ts / `landing` casefile-script.md·casefile-qa.md·WORKLOG.md·ARCHITECTURE.md

---

## 2026-05-30 01:00 — CTA 씬 효과 강화 + 시리즈 예고 제거 + sync-arch 작업로그 기능

**무엇을**
- CTAScene(`remotion/compositions/CaseFileVideo.tsx`) 효과 강화: 헤드 등장 spring + 펄스/글로우 호흡, "저장" 앰버 칩 + boxShadow 글로우 호흡, 카드 테두리 글로우, 댓글 칩 blink + 화살표 bob, 펄스 ring 2개. (밋밋함 해소, 저장·댓글 유도 시선 집중)
- 대본 룰에서 **시리즈 예고("다음편은 ~~편") 금지** 명문화 — `casefile-script.ts`(생성 룰8·QA hard-fail), `remotion/CLAUDE.md`, `.claude/commands/casefile-script.md`·`casefile-qa.md` 4곳 동시 갱신. "저장해두고 다시 보세요" 류로 대체.
- `/sync-arch` 스킬에 **WORKLOG.md 누적 기록 절차** 추가(타임스탬프·커밋 근거·append-only) → WORKLOG.md 신설·운영 시작.
- 운영: dev 서버 `.next` 캐시 손상(`ENOENT _document.js`, 프로덕션 빌드+dev 산출물 혼재) 복구 — 포트3000 종료·`.next` 삭제·재기동.

**왜**
- 회차가 유동적이라 "다음편" 약속을 지킬 수 없어 신뢰 리스크 → 룰로 영구 차단.
- CTA가 영상 마지막 전환 포인트인데 정적이라 저장·댓글 행동 유도가 약함.
- 문서 "현재 상태"만으로는 *어떤 과정을 거쳤는지* 추적 불가 → 시간순 히스토리 필요.

**영향 / 후속**
- CaseFileVideo.tsx 변경 — 로컬 렌더는 즉석 번들이라 즉시 반영. (Lambda 경로 쓸 일 있으면 재배포 필요하나 현재 로컬 전용이라 무관.)
- 룰 변경 3곳 동기화 규칙(코드 프롬프트/remotion CLAUDE.md/스킬 md) 준수함.
- 검증: case-002 실제 로컬 렌더로 QA 통과·CTA 나레이션 시리즈 예고 없음·효과 프레임 확인 완료.

**커밋**: `admin-tool` fa2986f / `landing` 2b98b68  (※ `.next` 복구는 운영 작업, 미커밋)

---

## 2026-05-30 00:34 — CaseFile 영상 로컬 렌더 전환 + 로컬 스튜디오 웹 패널 신설

**무엇을**
- 영상 렌더를 AWS Lambda → **완전 로컬**(`@remotion/bundler`+`@remotion/renderer`)로 전환. CLI `scripts/make-case.mts`(`npm run make:case`) 신설.
- QA 게이트: 검수 최종 실패 시 TTS 전에 `QAGateError` throw (`runCaseFilePipeline({ failOnQA })`).
- 폰트 최적화(`remotion/lib/fonts.ts`): weight(400/600/700/800/900)+subset(korean/latin) 제한 → 네트워크 요청 1116→605.
- 어드민 검수 UI(`review/page.tsx`)에서 영상 생성 버튼·폴링·롱폼 컬럼·음성 노브 제거 (검수/승인/스킵/편집만).
- **로컬 스튜디오 웹 패널** `/admin/local-studio` + API 4종(`app/api/local-studio/{cases,script,render,preview}`) 신설 — 케이스선택→설정→대본생성→편집(훅후보/before/after/diff)→로컬렌더→미리보기. 전부 `NODE_ENV=production`이면 403(dev 전용).

**왜**
- 배포된 Lambda serveUrl이 구형 번들(ShortsVideo만)이라 `Could not find composition CaseFileVideo` 발생. 로컬은 매번 즉석 번들 → 항상 최신 컴포지션. Lambda 비용·serveUrl 재배포 부담도 제거.
- 무거운 영상 작업은 로컬, 일상 운영(검수·큐레이션)은 온라인 어드민으로 환경 분리.

**영향 / 후속**
- 온라인 어드민에는 영상 기능 없음(403). 영상은 로컬에서만. `make-case.mts`와 웹 패널은 동일 파이프라인(`runCaseFilePipeline`) 재사용(SoT).
- Vercel hobby 플랜 maxDuration 상한(300)에 맞춰 render 라우트 600→300 수정(빌드 통과용, 프로덕션 미실행).
- 미검증: QA 통과 케이스의 happy-path 로컬 렌더(Claude+TTS 비용 때문에 수동 확인 대기).
- 확장 여지: 향후 온라인 영상 생성은 `enqueueCaseFileJob()` seam으로 큐 전환 가능(준비됨).

**커밋**: `admin-tool` 71d0d4b · f678d60 · 16c9a58 / `landing` 81680b9 · 5bb54a5

---
