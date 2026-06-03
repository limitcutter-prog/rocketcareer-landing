# 작업 로그 (WORKLOG)

> /sync-arch 실행 시 자동 누적. 최신 항목이 맨 위.
> 각 블록 = 한 작업 세션. 형식: 무엇을 / 왜 / 영향. 커밋 해시는 근거.

---

## 2026-06-03 17:06 — IMC 기획안 v1·v2 git 추적 시작

**무엇을**
- `marketing_division/` IMC 기획안 v1·v2를 git 추적 시작(그간 미추적). v2는 2026-06-03 sync 정합본(§1 인트로 4.4s·상단 헤더 태그라인, §12 2·3번 반영 완료).

**왜**
- 전략 문서가 코드와 함께 버전 관리되도록(백업·이력). ARCHITECTURE/WORKLOG와 동일 정책.

**영향 / 후속**
- 이후 IMC 변경도 `/sync-arch` 대상에 포함. (admin-tool의 `사용방법.txt` 리네임은 사용자 파일이라 미커밋 유지)

**커밋**: `landing` 41953d4

---

## 2026-06-03 17:03 — IMC v2 정합: CTA·서비스명(2번)·HR 화법(3번) + 인트로 최종(4.4s)

**무엇을**
- **인트로 최종화**: 5.0→**4.4s 순차**(셰브론 빠르게 발사·소멸 → "발사 준비 완료" 단독 → 로고+태그라인). 태그라인을 **상단 헤더로 이동**(CASE#·브랜드 아래), 하단 워터마크는 브랜드명.
- **2번 CTA·서비스명 IMC 정합**: `generatePublishKit` 고정댓글/설명에 "현직 HR이 직접 보는 **커리어 런칭 세션(무료 진단)** + 랜딩 링크"(HR멘토그룹) **명시**(전환 직결). 본문(영상)은 소프트 3톤 유지(이원화). 가드레일: 1회 담백·과장/압박/반복 금지(광고처럼 보이면 실패).
- **3번 HR 화법**: SYSTEM_PROMPT 룰 2d 신설 — 시점 HR내부→지원자(주어=상황·현상)·집단 목소리("저희가 수백 건…")·금지패턴("잘 쓰는 법 N가지"·"합격 보장"·HR 화자주어·방법론/툴 이름). QA hard-fail 추가.
- 룰 4곳 동기화(casefile-script.ts+QA / remotion CLAUDE.md / casefile-script.md / casefile-qa.md). v2 기획안 §1·§12 정정(미커밋, marketing_division 미추적).

**왜**
- v2 기획안(2026-06-03)과 코드 정합. 유튜브 유입 전환을 위해 고정댓글에서 무료 진단 명확 안내(과홍보는 가드레일로 통제). 영상 대본은 "현직 HR 집단" 화법으로 차별화.

**영향 / 후속**
- 본문 영상은 광고화 안 됨(소프트 유지). 다음 대본 생성/업로드 키트부터 적용.
- 후속: v2 doc을 git 추적할지 사용자 결정 대기(현재 미추적). BGM(음원 필요)은 별건.

**커밋**: `admin-tool` (이 커밋) / `landing` (이 sync 커밋)

---

## 2026-06-03 11:06 — 브랜드 인트로 V5 발사 심볼 확정 (5.0s 순차 연출)

**무엇을**
- 인트로 로켓을 **V5 추상 발사 심볼**(셰브론 3겹+스파크, SVG)로 교체. 색은 기존(네이비·그린·앰버) 유지.
- 5.0초 **순차 연출**: ① 셰브론 상승 발사(진동 rumble·덜컥 jolt·세로 스피드 스미어) → ② "당신의 커리어, 발사 준비 완료"(크게 강조 후 페이드) → ③ "로켓커리어연구소"(96px·샤인 스윕·글로우, 단독 1초+ 유지) → 아래 "현직 HR과 함께, 커리어 런칭 플랫폼"(글로우+샤인).
- 긴 태그라인은 인트로에서 빼고 **컨텐츠 하단 워터마크**로 상시 노출("현직 HR담당자와 함께, 커리어 런칭 플랫폼").
- `INTRO_SEC` 2.0→5.0, `totalCaseFrames`·씬 시작점에 반영(음성·자막 싱크 유지). 임시 시안 보드(`_RocketSiansBoard`)·Root 등록·테스트 렌더 제거.

**왜**
- 기존 로켓 디자인이 약해 V5로 확정. "로켓 발사=커리어 런칭" 브랜드 컨셉 강화. 사용자 피드백 반영(상승감·살아있는 진동·로고 노출 시간·태그라인 강조).

**영향 / 후속**
- 모든 CaseFileVideo 앞에 5.0s 인트로 추가(영상 길이 +5s). 렌더·TTS·구조 로직 무관.
- 검증: 스틸·mp4 프리뷰 반복 확인 완료. 진동 강도 사용자 요청대로 약화(rumble 9→4.5).
- **미구현(승인됨, 후속)**: 2번 CTA·서비스명 IMC 정합 / 3번 HR 화법 룰 — 다음에 코드+스킬 4곳 동기화 예정.

**커밋**: `admin-tool` 9c0c2d7 (branch feature/marketplace-phase1) / `landing` (이 sync 커밋)

---

## 2026-06-03 10:16 — 벤치마크 변칙 주입 + 브랜드 인트로 + IMC 정렬 결정

**무엇을**
- 트렌드 벤치마크 주입을 **변칙적**으로 재작성(랜덤 표본·랜덤 강조항목·리믹스 지시 회전) → 결과 획일화 차단. [커밋됨]
- 모든 영상 앞 **로켓 발사 브랜드 인트로(1.4s)** 추가(IntroScene, SVG·글리프안전, 기존 팔레트 유지). [커밋됨 — 로켓 모양/태그라인은 확정 대기]
- IMC 기획안(`marketing_division/로켓커리어연구소_IMC_마케팅기획안_v1.md`) 검토 → 영상 카피·컨셉 정렬 방향 도출.

**결정 (사용자 확정)**
- 🎨 **영상 팔레트는 기존 유지**(네이비 #1B2B4B / 그린 #27AE60 / 앰버 #FFC83D). IMC 브랜드북의 크림골드(#E8D5A3)는 **영상에 미사용 확정**(시안 V3·V4 폐기). 로켓은 색 유지 + **모양만** 변경.
- ✅ **CTA·서비스명 IMC 정합** 승인 — 본문은 소프트 유지, 고정댓글/설명은 "커리어 런칭 세션(무료)·HR멘토그룹·랜딩 링크"로. ("런칭"=로켓 발사 컨셉 시너지) → 구현 대기.
- ✅ **훅·나레이션 IMC 화법** 승인 — 시점 HR내부→지원자(주어=상황/현상), 집단 목소리("저희가 수백 건"), 금지패턴("~잘 쓰는 법 N가지"·"합격 보장"·HR 주인공) → 구현 대기.
- 캐치프레이즈(인트로 태그라인) 후보 압축 중: "현직 HR과 함께, 커리어 런칭" / "제대로 쏘아 올리는 커리어 런칭 플랫폼" / "당신의 커리어, 발사 준비 완료".

**영향 / 후속**
- 후속 커밋: 로켓 모양(시안 중 택1) + 태그라인 교체, CTA/화법 룰 4곳 동기화(코드·remotion CLAUDE.md·스킬 md 2개).
- 임시 디자인 도구 `remotion/compositions/_RocketSiansBoard.tsx`(+Root.tsx 등록)는 로켓 확정 후 제거.

**커밋**: `admin-tool` 45e670d (branch feature/marketplace-phase1) / `landing` (이 sync 커밋)

---

## 2026-06-01 20:52 — 내레이션 훅 생성 강화 (케이스 고유 + 다양한 유형)

**무엇을**
- 훅이 케이스마다 비슷하던 문제 해결. `casefile-script.ts` SYSTEM_PROMPT 룰10 전면 개편: 범용 문구("열심히 썼는데 왜 떨어질까요" 류) 기본값 금지, 그 케이스의 고유 디테일(직무·구체 실수·전환 수치·상황) 의무화.
- hook_candidates 3개를 **5가지 유형**(①충격 수치/결과 ②반전 ③구체 실수 지적 ④손실 강조 ⑤직무 특정 공감)으로 서로 다르게 생성.
- 룰3·유저프롬프트에서 클리셰 시드 제거 + "이 케이스의 가장 강력한 한 방으로 후킹" 지시. QA 관점3에 훅 케이스-고유성 점검 추가(약하면 revise가 개선).
- 동기화 4곳: 코드 SYSTEM_PROMPT / `remotion/CLAUDE.md` 룰8 / `.claude/commands/casefile-script.md`·`casefile-qa.md`.

**왜**
- 모든 영상 초기 훅이 거의 같아 후킹력 약함 → 케이스별로 강력하게 차별화.

**영향 / 후속**
- 대본 생성 룰만 변경(구조·렌더·TTS 무관). 다음 ③ 대본 생성부터 적용.
- ⚠️ 이 세션 외 다른 세션에서 멘토 포털·마켓플레이스 쇼케이스 작업 별도 진행됨(이 항목은 **훅 변경만** 기록). admin-tool 커밋은 `feature/marketplace-phase1` 브랜치 위에 올라감.

**커밋**: `admin-tool` 0ea3b39 (branch feature/marketplace-phase1) / `landing` bb4dd77

---

## 2026-05-30 12:32 — 유튜브 업로드 키트 (렌더 후 부가 ⑦)

**무엇을**
- 렌더(⑥) 후 `⑦ 업로드 키트 생성` 버튼 — 확정 대본으로 **제목 3안 / 설명란 / 랜딩 후킹 고정댓글 / 해시태그** 생성.
- `generatePublishKit`(casefile-script.ts) + 신규 라우트 `/api/local-studio/publish-kit`. UI는 편집 + 클립보드 복사. 제작 이력(`_studio-index.json`)에 kit 병합 저장 → 케이스 재선택 시 복원.
- 랜딩 URL: `NEXT_PUBLIC_LANDING_URL`(기본 `rocketcareer-landing.vercel.app`).

**왜**
- 영상 제작 후 유튜브 업로드(제목·설명·고정댓글)까지 한 화면에서 끝내고, 댓글로 랜딩 진단 유입을 후킹하려고. 기존 로직 무수정 부가 작업.

**영향 / 후속**
- ③~⑥ 플로우·렌더·TTS 무수정. local-studio API 7→8종(publish-kit 추가).
- 사용자 검증 완료(정상 작동, 랜딩 URL 확인). 후속: 실제 업로드해 제목·고정댓글 전환율 관찰.

**커밋**: `admin-tool` 8d8f4e8 / `landing` (이 sync 커밋)

---

## 2026-05-30 12:13 — 로컬 스튜디오 나레이션 우선 분리 플로우 + 다수 버그 수정

**무엇을**
- **제작 순서 분리(나레이션 우선)**: ③대본(나레이션) 생성 → 수정 → ④자막 생성(`generateSubtitlesFromNarration`, 나레이션 기준·before/after 원문 유지) → 수정 → ⑤음성(TTS, `generateCaseFileTTS`) → ⑥렌더. 파이프라인 `tts:false` 옵션·`/tts`·`/subtitles` 라우트 신설.
- **QA 권고 실제 반영**: `reviseCaseFileScript`로 검수 issues를 대본에 반영 후 노출(표시만 X).
- **제작 이력에 대본 저장**: `_studio-index.json`에 props·ttsScenes 저장 → 케이스 재선택 시 대본·자막·영상 복원.
- **버그 수정 3종**: ① preview Node스트림→Buffer 반환(미리보기 무한 스피너) ② TTS폴더·파일명 caseId(`caseSlug`) 기준(케이스 간 음성/영상 충돌) ③ JSON 출력 견고화(프리필 미지원 모델 대응 — 강한 JSON지시+1회 재시도).
- **dev 안정화**: webpack 메모리 캐시(`.next` 캐시 손상 방지) + `predev` 포트가드(`scripts/free-port.mjs`, 중복 dev 차단).

**왜**
- 자막·나레이션이 따로 생성돼 두 번 편집하는 번거로움 → 나레이션을 기준으로 자막이 따라오게.
- QA가 권고만 하고 안 고쳐 써먹지 못함 / case_number가 caseId와 어긋나 영상·음성이 다른 케이스로 섞임 / 중복 dev·캐시 손상으로 ENOENT 반복.

**영향 / 후속**
- 정상 `case-NNN` id는 파일명 동일 → 기존 영상 무영향. 충돌났던 stale 인덱스 항목 1건 정리함.
- `/admin/local-studio` API 6→7종(subtitles 추가). 사용자 실제 ③~⑥ 플로우 검증 완료(음성·자막·영상 일치 확인).
- 후속: 새 CTA 톤·분리 플로우로 영상 몇 편 만들며 줄바꿈·품질 체감.

**커밋**: `admin-tool` 66d938b / `landing` (이 sync 커밋)

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
