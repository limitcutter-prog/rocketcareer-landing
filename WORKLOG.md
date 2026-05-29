# 작업 로그 (WORKLOG)

> /sync-arch 실행 시 자동 누적. 최신 항목이 맨 위.
> 각 블록 = 한 작업 세션. 형식: 무엇을 / 왜 / 영향. 커밋 해시는 근거.

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
