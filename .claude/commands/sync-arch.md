# /sync-arch — 아키텍처 문서 동기화 + 작업 로그 기록

작업 완료 후 ① ARCHITECTURE.md와 실제 코드 상태가 맞는지 검증·업데이트하고,
② **이번 세션에 무슨 작업을 왜 했는지**를 일자·타임스탬프와 함께 `WORKLOG.md`에 누적 기록한다.

> 목적: 문서의 "현재 상태"만으로는 *어떤 과정을 거쳐 지금에 이르렀는지* 추적이 안 돼 혼선이 생긴다.
> sync 할 때마다 약식 작업 로그를 남겨 시간순 히스토리를 항상 트래킹한다.

## 토큰 절약 원칙 (필수) — 데이터 분리 저장

> 이력이 쌓이면 sync마다 토큰이 폭증한다. **현재 상태는 본문, 과거 이력은 아카이브**로 분리 저장한다.

- **ARCHITECTURE.md** = 현재 인벤토리·모듈 상세(본문)만. 변경 이력은 `ARCHITECTURE_CHANGELOG.md`로 분리. 본문 상단엔 `마지막 업데이트` 1줄 + `최근 변경` **3건 요약**만 둔다.
- **WORKLOG.md** = 최근 **5개 블록**만. 초과분은 `WORKLOG_ARCHIVE.md`로 이동.
- **읽기 규칙(sync 시) — 이게 토큰 핵심**:
  - `ARCHITECTURE_CHANGELOG.md`·`WORKLOG_ARCHIVE.md`는 **읽지 않는다**(특정 과거 이력이 필요할 때만 Grep/부분 Read).
  - `ARCHITECTURE.md`는 **통째로 읽지 말고** 변경 대상 모듈 행만 Grep(인벤토리 표·해당 모듈 섹션). 전체가 필요하면 Read offset/limit.
  - `WORKLOG.md`는 **head만**(`Read limit 40` ≈ 최근 블록 1~2개)로 직전 항목만 참조.
  - 큰 파일 갱신은 본문을 컨텍스트에 통째로 들이지 말고 Edit(고유 문자열 치환) 또는 스크립트로 처리. (UTF-8 주의: PowerShell `Get-Content`는 ANSI 기본이라 한글 깨짐 → 검증은 Read 도구나 `[System.IO.File]::ReadAllLines` 사용)

## 실행 절차

1. `ARCHITECTURE.md` 확인 — **통째 읽기 금지**: 변경 대상 모듈 행만 Grep(인벤토리 표·모듈 섹션). 변경 이력(CHANGELOG)·WORKLOG 아카이브는 읽지 않는다
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
4. 사용자 확인 후 ARCHITECTURE.md 갱신 — 아래 **"ARCHITECTURE 이력 절차"** 대로(CHANGELOG 분리 유지). 인벤토리 표·모듈 섹션·데이터 계약은 본문에서 직접 갱신.
5. 변경된 모듈이 있으면 해당 모듈 CLAUDE.md의 `현재 상태` 섹션도 업데이트
6. **작업 로그 기록 (필수)** — 아래 "작업 로그 절차" 수행

## ARCHITECTURE 이력 절차 (CHANGELOG 분리)

> `마지막 업데이트` 변경 이력은 **본문에 누적하지 않는다**(과거 그래서 한 줄이 1.4만 자까지 비대해짐). 전체 이력은 `ARCHITECTURE_CHANGELOG.md`.

1. 새 변경 이력 항목(상세 1단락)을 `ARCHITECTURE_CHANGELOG.md` **맨 위**(`---` 다음, 헤더 블록 아래)에 `## YYYY-MM-DD (**제목** — 상세…)` 형식으로 prepend. (아카이브 전체를 읽지 말고 head에만 삽입)
2. ARCHITECTURE.md 상단 두 줄을 교체:
   - `마지막 업데이트: YYYY-MM-DD — <이번 제목>`
   - `**최근 변경**` 목록을 **최신 3건**으로 유지(이번 항목을 맨 위에 추가하고 4번째는 삭제). 각 줄은 `- YYYY-MM-DD <제목>` 한 줄 요약만.
3. 상세 본문(인벤토리 상태·모듈 섹션·데이터 계약·마케팅 레이어)은 평소대로 본문에서 갱신.

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
3. `WORKLOG.md` head만 확인(`Read limit 40`, 직전 블록 참조용 — **전체 읽지 말 것**) 후 **최신 항목을 맨 위(헤더 `---` 다음)에 prepend**. 없으면 새로 생성(아래 헤더 포함).
4. 한 세션 = 한 블록. 약식이라도 **무엇을 / 왜 / 영향·후속**이 보이게. 커밋 해시를 근거로 첨부.
5. **로테이션(토큰 절약)**: prepend 후 WORKLOG.md 블록이 **5개 초과면 가장 오래된 블록을 `WORKLOG_ARCHIVE.md` 맨 위로 이동**(WORKLOG.md엔 최근 5개만 유지). 아카이브는 append-only·읽지 않음. 스크립트 권장(아래).

### WORKLOG.md 형식
```
# 작업 로그 (WORKLOG)

> /sync-arch 실행 시 자동 누적. 최신 항목이 맨 위. **최근 5개 블록만 유지**(초과분 → `WORKLOG_ARCHIVE.md`).
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
- 기존 항목은 절대 수정·삭제하지 않는다(append-only 히스토리). **아카이브로 이동은 삭제가 아님**(이력 보존). 오타 정정만 허용.
- 길게 쓰지 않는다. 블록당 8~12줄 이내 약식.

### 로테이션 스크립트 (WORKLOG 최근 5개 유지 → 초과분 아카이브)
> prepend 후 실행. UTF-8(no BOM)·LF로 기록. `Get-Content`(ANSI) 대신 `[IO.File]`로 읽고, 검증은 Read 도구 사용.
```powershell
$enc=New-Object Text.UTF8Encoding $false; $sep="`n`n---`n`n"; $keep=5
$p=(Resolve-Path WORKLOG.md).Path; $raw=([IO.File]::ReadAllText($p))-replace "`r",""
$parts=[regex]::Split($raw,'(?m)^[ \t]*---[ \t]*$')
$hdr=$parts[0].Trim(); $b=@($parts[1..($parts.Count-1)]|%{$_.Trim()}|?{$_})
if($b.Count -gt $keep){
  [IO.File]::WriteAllText($p,$hdr+$sep+(($b[0..($keep-1)])-join $sep)+"`n",$enc)
  $ap=(Resolve-Path WORKLOG_ARCHIVE.md).Path; $ar=([IO.File]::ReadAllText($ap))-replace "`r",""
  $ap2=[regex]::Split($ar,'(?m)^[ \t]*---[ \t]*$'); $ah=$ap2[0].Trim()
  $old=@($ap2[1..($ap2.Count-1)]|%{$_.Trim()}|?{$_}); $moved=$b[$keep..($b.Count-1)]
  [IO.File]::WriteAllText($ap,$ah+$sep+((@($moved)+@($old))-join $sep)+"`n",$enc)
}
```

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
