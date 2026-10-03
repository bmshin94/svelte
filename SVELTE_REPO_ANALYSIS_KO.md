# Svelte 레포지토리 전수조사 분석 리포트 (한글판) ✨

> 작성: 카리나 (Claude Code) 💖
> 작성일: 2026-10-03
> 분석 대상 버전: `svelte@5.57.0`

---

## 🔗 GitHub 주소

| 구분 | 주소 |
|---|---|
| **이 레포 (포크)** | https://github.com/bmshin94/svelte |
| **원본 (공식)** | https://github.com/sveltejs/svelte |
| **공식 사이트** | https://svelte.dev |
| **공식 문서** | https://svelte.dev/docs |
| **로드맵** | https://svelte.dev/roadmap |
| **Discord** | https://svelte.dev/chat |
| **RFC 레포** | https://github.com/sveltejs/rfcs |
| **Open Collective** | https://opencollective.com/svelte |

---

## 📑 목차

1. [이게 뭐하는 건가 — 정체 파악](#1-이게-뭐하는-건가--정체-파악)
2. [쉽게 설명 (비유 버전)](#2-쉽게-설명-비유-버전)
3. [질문 7개 답변](#3-질문-7개-답변)
4. [수익화 아이디어 완전판](#4-수익화-아이디어-완전판)
5. [부록 — 명령어 치트시트](#5-부록--명령어-치트시트)

---

## 1. 이게 뭐하는 건가 — 정체 파악

### 1.1 기본 정보

| 항목 | 내용 |
|---|---|
| 레포 이름 | `bmshin94/svelte` (개인 포크) |
| 원본 | `sveltejs/svelte` (Svelte 공식) |
| 버전 | `svelte@5.57.0` (Svelte 5 = 최신 메이저) |
| 라이선스 | **MIT** (상업적 이용/수정/재배포 자유) |
| 구조 | pnpm 모노레포 (`packages/*`, `playgrounds/*`) |
| 코드 규모 | JS 파일 368개 / 약 **64,479줄** |
| 패키지 매니저 | `pnpm@10.33.4` (필수) |
| Node 요구사항 | `>=18` |
| 작업 브랜치 | `claude/youthful-franklin-t70sa7` |
| 최근 커밋 | `4991cd4` — PR #1 머지 (CLAUDE.md 페르소나 가이드 추가) |

### 1.2 Svelte란?

React/Vue와 같은 "UI 구축 도구"지만, 작동 방식이 근본적으로 다르다.

```
[React / Vue 방식]
  내 코드 --> 브라우저에 "런타임 라이브러리"까지 함께 전송
         --> 브라우저에서 Virtual DOM 비교 연산 --> 실제 DOM 변경
  단점: 런타임이 무겁고, 비교 연산 비용 발생

[Svelte 방식]
  내 코드 --> **빌드 시점에 컴파일** --> 순수 바닐라 JS로 변환
         --> 브라우저는 "DOM을 콕 찍어 바꾸는 코드"만 실행
  장점: 런타임 거의 없음, 번들 작음, 빠름
```

- 공식 슬로건: **"Cybernetically enhanced web apps"**
- README 문구: **"web development for the rest of us"**

### 1.3 컴파일 예시

입력 (`.svelte`):

```svelte
<script>
  let count = $state(0);
</script>

<button onclick={() => count++}>
  {count} 번 클릭
</button>
```

컴파일 결과 (개념적 형태):

```js
const count = state(0);
const button = document.createElement('button');
button.onclick = () => set(count, get(count) + 1);
// count 가 바뀌면 이 텍스트 노드만 교체
template_effect(() => set_text(text, `${get(count)} 번 클릭`));
```

핵심: 브라우저에 "큰 라이브러리"가 가지 않고 **필요한 명령문만** 전달된다.

---

### 1.4 폴더 전수조사 결과

#### 루트

```
README.md              Svelte 소개
CONTRIBUTING.md        기여 가이드 (약 10KB, 매우 상세)
AGENTS.md         🤖   "AI 코딩 에이전트용 가이드" ← 중요
CLAUDE.md         💖   카리나 페르소나 가이드 (PR #1로 추가됨)
CODE_OF_CONDUCT.md     행동 규범
LICENSE.md             MIT
FUNDING.json           오픈소스 후원 정보
package.json           svelte-monorepo (private)
pnpm-workspace.yaml    packages/* + playgrounds/*
pnpm-lock.yaml         의존성 락 (약 150KB)
vitest.config.js       테스트 러너 설정
eslint.config.js       린트 설정
svelte.config.js       Svelte 설정
.prettierrc / .editorconfig / .npmrc / .gitattributes
vitest-xhtml-environment.ts
```

#### `.agents/` — AI 에이전트 전용 폴더 🤖

```
.agents/skills/performance-investigation/SKILL.md
```

Svelte 팀이 **AI 에이전트가 사용할 "스킬"을 레포에 직접 커밋**해 둔 것.

`SKILL.md` 내용 요약 (성능 회귀 조사 절차):

1. 측정할 브랜치에서 시작
2. `pnpm bench:compare main foo` 실행 (인자 1개면 자동으로 main과 비교)
3. 결과 확인 위치
   - 요약: `benchmarking/compare/.results/report.txt`
   - 원시 수치: `benchmarking/compare/.results/{main,branch}.json`
   - CPU 프로파일: `benchmarking/compare/.profiles/<branch>/*.cpuprofile`, `*.md`
4. 회귀가 큰 벤치마크부터 `.md` 프로파일 요약 비교
5. 런타임 내부 핫스팟 확인: `runtime.js`, `reactivity/batch.js`, `reactivity/deriveds.js`, `reactivity/sources.js`
6. 한 번에 하나씩 최적화 → 타깃 벤치 재실행
7. 수정 후 `pnpm test runtime-runes` 실행
8. 핫스팟 델타 확인: `node benchmarking/compare/profile-diff.mjs kairo_mux_owned main foo`

함정(gotchas):
- `bench:compare`는 브랜치를 자동 체크아웃하므로 **커밋 안 한 변경사항은 stash 필요**
- 실행마다 `.results`, `.profiles` 가 덮어쓰기됨

`AGENTS.md` 핵심 지침:
- `CONTRIBUTING.md` 도 반드시 읽어라
- PR 제출 시 `.github/PULL_REQUEST_TEMPLATE.md` 를 읽고 올바르게 작성하라
- **전체 테스트 스위트를 돌리지 않고 PR 제출 금지**
- 성능 조사 요청 시 `performance-investigation` 스킬을 사용하라

> 결론: 이 레포는 **AI가 기여하는 것을 전제로 설계된** 오픈소스다.

#### `packages/svelte/` — 본체 (npm `svelte` 패키지)

```
src/
├── compiler/                  🏭 컴파일러 (Svelte의 심장)
│   ├── phases/
│   │   ├── 1-parse/           ① .svelte --> AST (acorn 기반)
│   │   │   ├── index.js, acorn.js, remove_typescript_nodes.js
│   │   │   ├── read/, state/, utils/
│   │   ├── 2-analyze/         ② 분석 (스코프, 반응성 추적, CSS)
│   │   │   ├── index.js, visitors/, css/
│   │   ├── 3-transform/       ③ 변환 --> 코드 생성
│   │   │   ├── client/, server/, css/, shared/
│   │   ├── scope.js, nodes.js, bindings.js, patterns.js, css.js
│   ├── migrate/               Svelte 4 --> 5 자동 마이그레이션
│   ├── preprocess/            TS / SCSS / PostCSS 전처리 (플러그인 시스템)
│   ├── print/                 코드 출력
│   ├── errors.js / warnings.js / state.js
│   ├── validate-options.js / legacy.js
│   └── index.js               compile(), parse(), preprocess() 공개 API
│
├── internal/                  ⚙️ 런타임 (컴파일된 코드가 호출)
│   ├── client/
│   │   ├── reactivity/        ⭐ Signals 반응성 엔진
│   │   │   ├── sources.js        $state 의 실체
│   │   │   ├── deriveds.js       $derived 의 실체
│   │   │   ├── effects.js        $effect 의 실체
│   │   │   ├── batch.js          업데이트 일괄 처리 / 스케줄링
│   │   │   ├── async.js          비동기 반응성 (Svelte 5 신기능)
│   │   │   ├── props.js, store.js, equality.js, status.js, utils.js
│   │   ├── dom/               실제 DOM 조작
│   │   ├── dev/               개발 모드 전용 (경고/추적)
│   │   ├── runtime.js         런타임 코어
│   │   ├── proxy.js           Proxy 기반 객체 반응성
│   │   ├── hydratable.js      SSR 하이드레이션
│   │   ├── render.js, context.js, loop.js, timing.js
│   │   ├── error-handling.js, errors.js, warnings.js, validate.js
│   │   └── legacy.js
│   ├── server/                SSR 전용 런타임
│   ├── shared/                공용 코드
│   ├── flags/                 async / legacy / tracing 플래그
│   └── disclose-version.js
│
├── reactivity/                SvelteMap, SvelteSet, SvelteDate 등 반응형 내장객체
├── motion/                    spring, tweened
├── transition/                fade, fly, slide 등
├── animate/ easing/           애니메이션 유틸
├── store/                     Svelte 4식 스토어 (호환)
├── legacy/                    Svelte 4 호환 레이어
├── action/ attachments/ events/
├── server/                    서버 렌더링 공개 API
├── index-client.js            브라우저 엔트리
├── index-server.js            서버 엔트리
├── index.d.ts / ambient.d.ts  타입 정의
├── constants.js / utils.js / escaping.js / version.js
└── html-tree-validation.js

tests/                         🧪 테스트 42개 디렉토리
├── runtime-runes/             Svelte 5 룬 런타임
├── runtime-legacy/            Svelte 4 호환
├── runtime-browser/           실제 브라우저 (Playwright)
├── runtime-production/        프로덕션 빌드
├── runtime-xhtml/             XHTML 모드
├── compiler-errors/           에러 메시지 검증
├── validator/                 검증 로직
├── parser-modern/ parser-legacy/
├── server-side-rendering/ hydration/
├── css/ css-parse.test.ts / sourcemaps/
├── signals/ snapshot/ print/
├── migrate/ preprocess/ motion/ store/ types/
└── suite.ts / helpers.js / html_equal.js / animation-helpers.js

messages/                      에러/경고 메시지를 마크다운으로 관리 --> 코드 자동생성
elements.d.ts (약 87KB)        모든 HTML 요소 타입 정의
svelte-html.d.ts               svelte:element 타입
CHANGELOG.md (약 262KB)        Svelte 5 변경 이력
CHANGELOG-pre-5.md (약 182KB)  Svelte 4 이전 이력
rollup.config.js               compiler 번들 빌드
scripts/                       코드 생성 스크립트
knip.json                      미사용 코드 검출 설정
```

#### `documentation/docs/` — svelte.dev 공식 문서 원본

```
01-introduction/   개요, 시작하기, .svelte 파일, .svelte.js 파일
02-runes/          ⭐ $state, $derived, $effect, $props, $bindable, $inspect, $host
03-template-syntax/  {#if}, {#each}, {#await}, {#snippet} 등
04-styling/        <style>, 스코프 CSS, :global
05-special-elements/ <svelte:window>, <svelte:head> 등
06-runtime/        스토어, 컨텍스트, 라이프사이클
07-misc/           기타 (+ .generated)
98-reference/      API 레퍼런스 (+ .generated)
99-legacy/         Svelte 4 레거시 문서
```

> svelte.dev에 게시되는 문서 전체가 마크다운으로 들어있다. 번역/강의 자료의 원천.

#### `benchmarking/` — 성능 측정 시스템

```
benchmarks/
├── reactivity/tests/   kairo_* 벤치마크 13종 (반응성 성능 업계 표준)
│   clean_effects, kairo_avoidable, kairo_broad, kairo_broad_block,
│   kairo_deep, kairo_deep_block, kairo_diamond, kairo_mux,
│   kairo_repeated, kairo_triangle, kairo_unstable, mol, repeated_deps
├── reactivity/sbench.js, util.js
├── compiler/           parser.bench.js (파서 성능)
└── ssr/                서버렌더링 성능
compare/                브랜치 간 비교 + profile-diff.mjs
run.js / profile-compiler.js / analyze-compiler-profile.js
compiler-profiling.md
```

#### `playgrounds/sandbox/` — 로컬 실험장

Vite 기반. 컴파일러를 수정하고 즉시 눈으로 확인하는 곳.

```
index.html / demo.css / run.js / vite.config.js / svelte.config.js
ssr-common.js / ssr-dev.js / ssr-prod.js
scripts/create-app-svelte.js, create-test.js, download.js, hash.js, main.template.svelte
```

#### `.github/`

```
workflows/ci.yml                     테스트 자동 실행
workflows/release.yml                npm 자동 배포 (changeset)
workflows/autofix.yml                린트 자동 수정
workflows/ecosystem-ci-trigger.yml   생태계 호환성 검증
ISSUE_TEMPLATE/bug_report.yml, feature_request.yml, config.yml
ISSUE_TEMPLATE.md / PULL_REQUEST_TEMPLATE.md / FUNDING.yml
```

#### 기타

- `.changeset/` — 버전/체인지로그 자동 관리
- `.well-known/` — 도메인 검증용
- `.vscode/` — 에디터 설정
- `assets/` — 배너 이미지 (banner.png, banner_dark.png)

---

### 1.5 이건 어떨 때 쓰는 것인가

중요: 이 레포는 **"Svelte를 사용하기 위한 것"이 아니라 "Svelte 자체를 개발하기 위한 것"** 이다.

```
앱 만들려고 다운로드?       --> ❌ 아니다. 그럴 땐 `npx sv create my-app`
Svelte 내부를 고치거나 배우려고? --> ✅ 이게 그것이다
```

구체적 용도:

1. **Svelte 오픈소스 기여** — 버그 수정 PR (`good first issue` 라벨부터)
2. **컴파일러 내부 학습** — "$state는 어떻게 동작하나?" --> `sources.js` 확인
3. **성능 최적화 연구** — `.agents/skills/performance-investigation` 그대로 수행
4. **커스텀 Svelte 포크** — MIT라서 수정 배포 가능
5. **AI 에이전트 설계 레퍼런스** — `AGENTS.md` + `.agents/skills/` 패턴
6. **문서 번역 / 강의 제작** — `documentation/` 활용

### 1.6 어떤 도움이 되는가

| 순위 | 가치 | 설명 |
|:-:|---|---|
| ⭐⭐⭐⭐⭐ | **AI 에이전트 설계 교본** | `AGENTS.md` + `SKILL.md` = AI에게 레포 작업법을 가르치는 표준 패턴. 복사해서 자기 프로젝트에 적용 가능 |
| ⭐⭐⭐⭐⭐ | **AI 워크플로우 컨설팅 자산** | "세계 최정상 OSS 팀의 실제 패턴"이라는 레퍼런스 |
| ⭐⭐⭐⭐ | **포트폴리오 / 커리어** | PR 1건 머지 = 스타 13만 프로젝트 컨트리뷰터 이력 |
| ⭐⭐⭐⭐ | **프론트엔드 실력 레벨업** | 컴파일러 3단계 구조, Signals 반응성, Proxy, SSR+하이드레이션 |
| ⭐⭐⭐ | **콘텐츠 금광** | `documentation/`(커리큘럼), `CHANGELOG.md`(연재), `messages/`(트러블슈팅) |
| ⭐⭐⭐ | **비즈니스 자유도** | MIT 라이선스 = 상업적 이용 가능 |

### 1.7 한 줄 요약

> **"웹 프레임워크 Svelte 5의 전체 소스코드 + AI 에이전트가 기여하는 법까지 설계된, 살아있는 최고급 오픈소스 교재"**

---

## 2. 쉽게 설명 (비유 버전)

### 2.1 라면 비유

```
🍜 React / Vue 방식 = 「밀키트」
   손님에게 재료 + 레시피 + 조리도구를 함께 보냄
   --> 손님(브라우저)이 직접 요리
   --> 택배 상자가 무겁고(번들 큼), 요리 시간 필요(런타임 계산)

🍜 Svelte 방식 = 「완성된 라면 배달」
   주문받으면 우리 주방(빌드 서버)에서 미리 조리
   --> 손님은 포크만 들면 됨
   --> 상자 가볍고(번들 작음), 바로 먹음(빠름)
```

"우리 주방" = 컴파일러. 이 폴더는 **그 주방의 설계도 전체**.

### 2.2 공장 비유

```
📦 받은 것 = 「Svelte 라면 공장 설계도 + 공장 전체」

packages/svelte/src/compiler/   🏭 1층: 가공 공장
   1-parse    --> 글자 읽기     ("<button>" 이 무엇인지 파악)
   2-analyze  --> 생각하기      ("이 변수 바뀌면 여기 다시 그려야 함")
   3-transform--> 코드 뽑기     (실제 JS 코드로 출력)

packages/svelte/src/internal/   ⚙️ 2층: 부품 창고
   reactivity/sources.js   $state 의 몸통
   reactivity/deriveds.js  $derived 의 몸통
   reactivity/effects.js   $effect 의 몸통
   dom/                    실제로 화면을 바꾸는 손

packages/svelte/tests/          🧪 3층: 품질검사실 (42개 라인)
benchmarking/                   ⚡ 4층: 속도 측정실
documentation/                  📖 5층: 사용설명서 원본
playgrounds/sandbox/            🎮 옥상: 시험운전장
.agents/skills/                 🤖 로봇 직원 교육 매뉴얼
```

### 2.3 자동차 비유

```
자동차 운전 (앱 개발)        vs    자동차 공장 견학 (이 레포)
= npx sv create my-app              = pnpm install && pnpm test
```

### 2.4 3줄 요약

1. 🤖 **AI 자동화 설계도**: `AGENTS.md` + `.agents/skills/` 구조를 복사해서 활용
2. 📈 **실력 + 경력**: PR 1개 머지로 대형 OSS 컨트리뷰터 이력 확보
3. 💰 **콘텐츠/비즈니스 자산**: MIT + 공식 문서 원본 = 강의·블로그·SaaS 전부 가능

---

## 3. 질문 7개 답변

### Q1. 설치 및 사용법?

#### 케이스 A: "Svelte로 앱 만들고 싶다" (일반적인 경우)

이 폴더는 필요 없다.

```bash
npx sv create my-app      # 공식 생성기 (SvelteKit 포함 선택 가능)
cd my-app
npm install
npm run dev               # --> http://localhost:5173
```

#### 케이스 B: "이 소스코드 자체를 돌려보고 싶다"

```bash
# 0. 전제조건
node -v        # 18 이상 필요
npm i -g pnpm  # npm/yarn 아니고 pnpm 필수 (10.33.4 권장)

# 1. 의존성 설치 (모노레포 전체)
cd /path/to/svelte
pnpm install

# 2. 빌드
pnpm build                 # packages/* 전부 빌드

# 3. 테스트
pnpm test                  # 전체
pnpm test runtime-runes    # Svelte 5 반응성만
pnpm test validator        # 검증 로직만

# 4. 타입체크 + 린트
pnpm check
pnpm lint
pnpm format                # 자동 정리

# 5. 실험장 띄우기
cd playgrounds/sandbox
pnpm dev                   # 컴파일러 수정 --> 즉시 확인

# 6. 성능 벤치마크
pnpm bench                                # 전체
pnpm bench kairo_mux kairo_deep           # 특정 것만
pnpm bench:compare main my-branch         # 브랜치 비교
pnpm bench:debug                          # 디버거 연결
pnpm profile:compiler                     # 컴파일러 프로파일링
```

#### 결과물 위치

| 파일 | 내용 |
|---|---|
| `benchmarking/compare/.results/report.txt` | 성능 비교 요약 |
| `benchmarking/compare/.results/*.json` | 원시 수치 |
| `benchmarking/compare/.profiles/*/*.md` | CPU 핫스팟 요약 (가장 유용) |

#### 함정

- `bench:compare`는 브랜치를 자동 체크아웃 → 커밋 안 한 변경사항은 `git stash` 필요
- 실행마다 `.results`, `.profiles` 덮어쓰기

#### 기여(PR) 워크플로우

```bash
git checkout -b fix/my-bug
# ... 코드 수정 ...
pnpm test                          # AGENTS.md: 전체 테스트 없이 PR 금지
npx changeset                      # 버전 변경 기록 추가 (필수)
git commit -m "fix: ..."
git push -u origin fix/my-bug
# --> PULL_REQUEST_TEMPLATE.md 양식 작성 후 PR
```

---

### Q2. 플러그인인가, 스킬인가, MCP인가?

**정답: 셋 다 아니다.** 이건 **"프레임워크 소스코드 레포지토리"** 다.

단, **그 안에 스킬이 1개 들어있다.**

```
bmshin94/svelte  <-- 전체는 그냥 "오픈소스 레포"
└── .agents/skills/performance-investigation/SKILL.md
                         ^-- 이것만 진짜 "스킬"
```

#### 3가지 개념 비교

| | **MCP** | **스킬 (Skill)** | **플러그인 (Plugin)** |
|---|---|---|---|
| 정체 | 프로토콜 (규격) | 마크다운 지침서 | 묶음 패키지 |
| 비유 | 🔌 USB 규격 | 📋 업무 매뉴얼 | 📦 종합 선물세트 |
| 하는 일 | AI ↔ 외부 시스템 **연결** | AI에게 **방법 교육** | 스킬+명령+훅 **묶어 배포** |
| 실행 | 서버 프로세스 상주 | 실행 안 됨, 읽히는 문서 | 설치형 |
| 예시 | GitHub MCP, Slack MCP | `performance-investigation` | 사내 워크플로우 팩 |
| 파일 | `mcp.json` 설정 | `SKILL.md` | `plugin.json` + 폴더 |
| 토큰 필요 | 보통 필요 | 불필요 | 경우에 따라 |

#### SKILL.md 구조

```markdown
---
name: performance-investigation
description: Investigate performance regressions and find opportunities for optimization
---

## Quick start
1. 측정할 브랜치에서 시작
2. pnpm bench:compare main foo 실행
...
```

= **YAML 프론트매터(이름+설명) + 마크다운 본문**. 이것이 스킬의 전부.

#### `.agents/` vs `.claude/`

```
.agents/skills/     여러 AI 도구가 공용으로 읽는 위치 (AGENTS.md 생태계)
.claude/skills/     Claude Code 전용 위치
```

Svelte 팀은 특정 AI에 종속되지 않도록 `.agents/`를 선택했다.

#### 보너스: 컴파일러 플러그인 시스템

```
packages/svelte/src/compiler/preprocess/
```

TypeScript, SCSS, PostCSS 등을 `.svelte` 파일에서 쓸 수 있게 하는 **전처리기 플러그인 인터페이스**. (AI 플러그인이 아니라 컴파일러 플러그인)

---

### Q3. API 토큰을 사용해야 하는가?

**Svelte 자체는 토큰이 전혀 필요 없다.**

```
pnpm install / build / test / bench / dev
--> 전부 토큰 0개, 완전 로컬 (install 때만 npm 레지스트리 접근)
```

Svelte는 빌드 도구이지 클라우드 서비스가 아니다.

#### 상황별 토큰 필요 여부

| 하려는 일 | 토큰 | 설명 |
|---|:---:|---|
| 소스 받기/빌드/테스트 | ❌ | 불필요 |
| 벤치마크/프로파일링 | ❌ | 전부 로컬 |
| 샌드박스 개발서버 | ❌ | 로컬 Vite |
| 내 포크에 push | ✅ | GitHub PAT or SSH키 |
| PR 생성/머지 | ✅ | GitHub 토큰 |
| CI(GitHub Actions) | ⚙️ | `GITHUB_TOKEN` 자동 주입 |
| npm publish | ✅ | npm 토큰 — **공식 메인테이너만** |
| Claude Code 사용 | ✅ | Claude 구독 or Anthropic API 키 |
| 직접 만들 AI 서비스 | ✅ | Anthropic/OpenAI 등 API 키 |

#### 토큰 보안 수칙

```bash
# 금지
const KEY = "sk-ant-xxxxx";   // 코드에 하드코딩 금지

# 올바른 방법
# .env 파일에 저장 + .gitignore 에 .env 추가
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

- **서버사이드에서만 사용** (브라우저 코드에 넣으면 노출됨)
- SvelteKit이면 `$env/dynamic/private` 또는 `$env/static/private` 사용

---

### Q4. AI 에이전트 구축에 도움이 될까?

**매우 도움된다. 두 가지 다른 방향으로.**

#### 4-1. 간접적 도움 ⭐⭐⭐⭐⭐ — "에이전트 설계 패턴 교본"

**패턴 1: 계층형 지침 구조 (Progressive Disclosure)**

```
AGENTS.md (얇은 입구)
   ├─ "CONTRIBUTING.md도 읽어라"      --> 기존 문서 재활용
   ├─ "PR 템플릿 꼭 채워라"            --> 출력 형식 강제
   ├─ "전체 테스트 없이 PR 금지"        --> 품질 게이트
   └─ "성능 조사면 이 스킬 써라"        --> 조건부 스킬 로딩
         |
   .agents/skills/performance-investigation/SKILL.md (깊은 전문지식)
```

이유: 모든 지식을 항상 로딩하면 컨텍스트가 터진다. 필요할 때만 꺼내 쓰는 구조.

**패턴 2: 스킬 안에 "검증 루프"를 내장**

```
SKILL.md가 제공하는 것:
  1. 무슨 명령 (pnpm bench:compare)
  2. 결과 위치 (report.txt 경로)
  3. 무엇을 볼지 (regression 큰 것부터)
  4. 어디를 의심 (runtime.js, batch.js, deriveds.js, sources.js)
  5. 무엇을 돌릴지 (pnpm test runtime-runes)
  6. 함정 (브랜치 체크아웃 주의)
```

= "측정 → 분석 → 수정 → 재검증" 사이클이 문서 하나로 완결. 에이전트가 혼자 끝까지 갈 수 있다.

**패턴 3: 에이전트 친화적 레포 환경**

| 요소 | 에이전트에게 좋은 이유 |
|---|---|
| 테스트 42개 디렉토리 | 변경 검증을 스스로 수행 가능 |
| `pnpm test runtime-runes` 부분 실행 | 빠른 피드백 루프 |
| `messages/` 마크다운→코드 생성 | 에러 메시지 수정이 쉬움 |
| `.changeset/` | 버전 기록 자동화 |
| CI/autofix 워크플로우 | 린트는 봇이 처리 |
| `playgrounds/sandbox` | 실제 동작 확인 가능 |

액션 아이템:

```bash
mkdir -p .agents/skills/my-deploy
# AGENTS.md 작성 (얇게, 입구 역할)
# .agents/skills/my-deploy/SKILL.md (명령 + 경로 + 판단기준 + 함정)
```

#### 4-2. 직접적 활용 ⭐⭐⭐⭐ — "에이전트 UI 만들기 최적 도구"

| 에이전트 UI 요구사항 | Svelte 5 솔루션 |
|---|---|
| 토큰 스트리밍 표시 | `$state` + 세밀한 업데이트 → 리렌더 0, 60fps |
| 도구 호출 진행상황 | `$derived` 로 파생상태 자동계산 |
| 다중 에이전트 상태 | `SvelteMap` (반응형 Map) |
| 비동기 대기 | Svelte 5 async 반응성 (`async.js`) |
| 번들 크기 | 런타임 거의 없음 → 모바일/임베드 유리 |
| SSR + 스트리밍 | `index-server.js` + SvelteKit |

```svelte
<!-- 에이전트 채팅 UI 예시 -->
<script>
  let messages = $state([]);
  let streaming = $state('');
  let tokenCount = $derived(streaming.length);

  async function ask(q) {
    const res = await fetch('/api/agent', {
      method: 'POST', body: JSON.stringify({ q })
    });
    const reader = res.body.getReader();
    const dec = new TextDecoder();
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      streaming += dec.decode(value);  // 이 한 줄로 화면 갱신
    }
    messages = [...messages, { role: 'assistant', text: streaming }];
    streaming = '';
  }
</script>

{#each messages as m}
  <div class={m.role}>{m.text}</div>
{/each}
{#if streaming}<div class="typing">{streaming}</div>{/if}
<small>{tokenCount} chars</small>
```

#### 4-3. 심화 활용 ⭐⭐⭐ — 컴파일러를 에이전트 도구로

```js
import { compile } from 'svelte/compiler';

// AI가 생성한 Svelte 코드를 검증하는 도구
function validateAgentOutput(code) {
  try {
    const { js, warnings } = compile(code, { generate: 'client' });
    return { ok: true, warnings };           // AI에게 경고 피드백
  } catch (e) {
    return { ok: false, error: e.message };  // AI에게 에러 피드백 --> 재시도
  }
}
```

= "AI가 UI 코드 생성 → 컴파일러로 즉시 검증 → 실패하면 에러 넘겨 자동 수정" 루프.

#### 정리

| 활용 방향 | 점수 | 요약 |
|---|:---:|---|
| 에이전트 레포 설계 패턴 학습 | ⭐⭐⭐⭐⭐ | 지금 당장 복사해서 사용 |
| 에이전트 UI 프론트엔드 | ⭐⭐⭐⭐ | 스트리밍 UI 최강 |
| 컴파일러를 검증 도구로 | ⭐⭐⭐ | AI 코드생성 서비스 핵심 |
| 에이전트 백엔드/오케스트레이션 | ⭐ | Python/Node + SDK 영역 |

---

### Q5. 수익화 아이디어가 있는가?

→ [4장 수익화 아이디어 완전판](#4-수익화-아이디어-완전판) 참조.

---

### Q6. React나 PHP로 만들 수 있는가?

"무엇을 만들지"에 따라 답이 다르다.

| 만들 것 | React | PHP | 평가 |
|---|:---:|:---:|---|
| Svelte 같은 컴파일러를 처음부터 | 🟡 가능하나 거대 | 🔴 비추천 | 수년 작업. 학습용 축소판은 가능 |
| **Signals 반응성 엔진만** | 🟢 **추천** | 🟡 의미 적음 | 주말 프로젝트로 적합 ⭐ |
| Svelte 교육 사이트 (한글) | 🟢 가능 | 🟢 가능 | 수익화 1순위 |
| 온라인 Svelte 플레이그라운드 | 🟢 React로 UI | 🟡 PHP는 서버만 | `svelte/compiler`는 JS → Node 필요 |
| AI → Svelte 코드 생성 SaaS | 🟢 React UI 가능 | 🟢 PHP 백엔드 가능 | 컴파일 검증은 Node 마이크로서비스 |
| Svelte 컴포넌트 마켓플레이스 | 🟢 | 🟢 (Laravel 적합) | 플랫폼은 자유 |

#### 1) Signals 반응성 엔진 직접 만들기 — 가장 추천 ⭐⭐⭐⭐⭐

`sources.js` / `deriveds.js` / `effects.js` 핵심 아이디어를 100줄로 재현:

```js
// mini-signals.js — Svelte 반응성의 핵심 원리
let activeEffect = null;

export function source(value) {
  const subs = new Set();
  return {
    get value() {
      if (activeEffect) {                 // 자동 의존성 추적
        subs.add(activeEffect);
        activeEffect.deps.add(subs);
      }
      return value;
    },
    set value(v) {
      if (Object.is(value, v)) return;    // 같으면 무시 (equality.js 원리)
      value = v;
      for (const fn of [...subs]) fn.run();  // 구독자만 실행
    }
  };
}

export function effect(fn) {
  const e = {
    deps: new Set(),
    run() {
      for (const d of e.deps) d.delete(e);  // 이전 의존성 정리
      e.deps.clear();
      const prev = activeEffect;
      activeEffect = e;
      try { fn(); } finally { activeEffect = prev; }
    }
  };
  e.run();
  return () => { for (const d of e.deps) d.delete(e); };
}

export function derived(fn) {
  const s = source(undefined);
  effect(() => { s.value = fn(); });       // derived = effect + source
  return { get value() { return s.value; } };
}
```

사용:

```js
const count = source(0);
const double = derived(() => count.value * 2);
effect(() => console.log(`${count.value} --> ${double.value}`));
// 출력: 0 --> 0
count.value = 5;
// 출력: 5 --> 10   (자동)
```

React 연동 훅:

```jsx
function useSignal(signal) {
  const [, force] = useReducer(x => x + 1, 0);
  useEffect(() => effect(() => { signal.value; force(); }), [signal]);
  return signal.value;
}
```

→ "반응성 엔진 직접 구현" 포트폴리오 완성.

#### 2) 컴파일러를 PHP로?

기술적으로는 가능하지만:

| 문제 | 설명 |
|---|---|
| 🔴 생태계 단절 | Vite/Rollup/esbuild 전부 JS → PHP가 끼어들 자리 없음 |
| 🔴 JS 파서 부재 | `<script>` 내부 JS 파싱에 acorn 급 도구가 PHP엔 없음 |
| 🔴 유지보수 부담 | Svelte는 지금도 매주 업데이트 |
| 🟢 대안 | PHP라면 **Laravel Blade + Livewire/Alpine** 조합이 정답 |

PHP의 올바른 활용법:

```
[Laravel/PHP]  -->  REST API, DB, 인증, 결제, 관리자
      | JSON
[Svelte/SvelteKit]  -->  프론트엔드 (SPA 또는 SSR)
```

이 조합이 실무에서 가장 많이 쓰이는 궁합이다.

#### 3) React vs Svelte 판단 가이드

```
Svelte를 "배우는 중"      --> Svelte로 만들기 (복습 효과)
React에 "익숙"           --> React로 빨리 만들고 Svelte는 학습용
교육 콘텐츠 제작          --> Svelte (신뢰도 상승)
회사 프로젝트            --> 팀 스택 따르기
```

---

### Q7. 유튜브 강의 영상으로 제작 가능할까?

**가능한 수준이 아니라 "콘텐츠 금광"이다.**

#### 왜 유리한가

| 강점 | 설명 |
|---|---|
| 📖 대본이 이미 있음 | `documentation/docs/` 전체가 공식 커리큘럼 |
| ⚖️ MIT 라이선스 | 코드 화면에 띄우고 설명 → 법적으로 안전 |
| 🇰🇷 한글 콘텐츠 희소 | "Svelte 5 룬" 한글 심화강의 거의 없음 = 블루오션 |
| 🎬 시각적으로 강렬 | "코드 → 컴파일 결과" 비교 화면 |
| 🧠 차별화 소재 | 남들은 사용법만, 여기는 **내부 구현**까지 |
| 🤖 AI 트렌드 결합 | `.agents/skills` 소재 = 현재 가장 핫한 주제 |

#### 추천 커리큘럼

**시리즈 A: "Svelte 5 입문" (초급, 조회수용) — 8편**

```
EP1  Svelte가 React보다 빠른 이유 (컴파일 vs 런타임)   8분
EP2  5분만에 첫 앱 (npx sv create)                    5분
EP3  $state 완전정복                                10분
EP4  $derived — 계산된 값                            8분
EP5  $effect — 부작용과 함정                        12분
EP6  $props & $bindable — 컴포넌트 통신              10분
EP7  전환효과 & 애니메이션 (transition/motion)        10분
EP8  SvelteKit으로 배포까지                          15분
```
소스: `documentation/docs/01-introduction`, `02-runes`, `03-template-syntax`

**시리즈 B: "Svelte 내부 해부" (고급, 차별화) — 7편**

```
EP1  .svelte 파일이 JS로 변하는 과정 전체 추적        20분
EP2  1-parse: AST 만들기 (acorn 활용)               18분
EP3  2-analyze: 반응성을 어떻게 알아내나             20분
EP4  3-transform: 코드 생성의 비밀                   20분
EP5  sources.js 해부 — $state의 진짜 모습           25분  ⭐킬러
EP6  deriveds/effects/batch — 업데이트 스케줄링      25분
EP7  Proxy로 객체 반응성 만들기 (proxy.js)           20분
```
소스: `packages/svelte/src/compiler/phases/`, `src/internal/client/reactivity/`

**시리즈 C: "오픈소스 기여 + AI 자동화" (실전, 수익화 연결) — 6편**

```
EP1  스타 13만 레포에 PR 날리기 (good first issue)    15분
EP2  테스트 구조 읽는 법 (42개 디렉토리 탐험)          15분
EP3  성능 벤치마크 실전 (pnpm bench:compare 라이브)    20분
EP4  CPU 프로파일로 병목 찾기                        18분
EP5  AGENTS.md + SKILL.md — AI에게 일 시키는 법      20분  ⭐⭐
EP6  Claude Code로 Svelte에 실제 기여하기 (라이브)     25분  ⭐
```

#### 제작 실전 팁

지킬 것:

```
1. 코드 화면에 출처 표기: "출처: github.com/sveltejs/svelte (MIT License)"
2. 로고/배너(assets/) 사용은 상표권 영역 --> "비공식 강의" 명시
3. 공식 문서 번역 시 "내 설명 + 예제" 추가 (단순 번역은 가치 낮음)
4. 버전 명시 필수 (Svelte 5.57 기준) — 룬은 계속 진화 중
```

영상 구성 공식:

```
0:00  훅 (결과 먼저 — "이게 1KB로 됩니다")
0:15  문제 제시 ("React에서 이거 왜 느릴까요?")
1:00  본론 (코드 + 터미널 분할화면)
       왼쪽: 작성한 .svelte
       오른쪽: 컴파일된 JS   <-- 핵심 화면
8:00  정리 + 다음편 예고
```

제목 예시:

```
"React 개발자가 Svelte 5 써보고 충격받은 이유"
"$state 안을 열어봤습니다 (Svelte 소스코드 해부)"
"AI가 오픈소스에 기여하는 시대 — Svelte팀의 AGENTS.md 분석"
"번들 크기 90% 줄인 프레임워크의 비밀"
"Signals, 100줄로 직접 만들어봤습니다"
```

#### 현실적 기대치

| 시리즈 | 조회수 | 수익성 | 난이도 |
|---|:---:|:---:|:---:|
| A (입문) | 🟢 높음 | 🟡 광고 위주 | 🟢 쉬움 |
| B (내부해부) | 🟡 중간 | 🟢 유료강의 전환 최고 | 🔴 어려움 |
| C (기여+AI) | 🟢 높음 | 🟢 컨설팅/강의 유입 | 🟡 보통 |

추천 전략:

```
1단계: A 시리즈로 구독자 확보 (유입)
2단계: C-EP5,6 (AI 소재)로 바이럴
3단계: B 시리즈를 유료 강의로 패키징 (수익)
```

---

## 4. 수익화 아이디어 완전판

### TIER 1 — 즉시 시작 가능 (초기비용 0원)

#### #1. 한글 Svelte 5 유료 강의 ⭐⭐⭐⭐⭐

```
상품     : 온라인 강의 (인프런 / 패스트캠퍼스 / 자체 플랫폼)
가격     : 5~15만원
수익모델  : 인프런 수익배분(약 50~70%) or 자체 100%
제작기간  : 2~3개월
타겟     : React 쓰다 지친 주니어~중급 개발자
```

성립 근거:
- 한글 Svelte 5 심화강의 거의 없음 = 선점 효과
- 대본이 `documentation/`에 이미 존재 → 제작 속도 3배
- "컴파일러 내부까지" = 완전 차별화

실행 로드맵:

```
1주차  유튜브에 무료 EP1~3 올려서 반응 테스트
2주차  댓글/수요 확인 --> 커리큘럼 확정
1개월  시리즈 A(입문) 완성 --> 무료 공개 (유입용)
2개월  시리즈 B(내부해부) 제작 --> 유료 패키징
3개월  인프런 등록 + 유튜브에서 랜딩 유도
```

예상: 수강생 200명 x 8만원 x 60% = **약 960만원** (첫 해, 가정 기반 추정)

#### #2. 유튜브 채널 + 애드센스/스폰서 ⭐⭐⭐⭐

```
상품     : "프레임워크 내부 해부" 전문 채널
수익원   : 애드센스 + 멤버십 + 기업 스폰서 + 강의 유입
수익화시점 : 구독 1,000명 + 4,000시간 (약 6개월~1년)
```

전략:
- "사용법" 채널은 포화 → **"내부 구현"** 포지셔닝
- 쇼츠: "컴파일 전 vs 후 코드 비교" 15초 영상
- `.agents/skills` AI 소재 = 최고 화제성

시리즈 B는 조회수는 낮아도 신뢰를 주어 강의·컨설팅 전환율이 높다.

#### #3. 기술 블로그 + 뉴스레터 ⭐⭐⭐

```
상품     : Svelte/Signals 심화 뉴스레터
수익원   : 유료 구독(월 5천원) + 스폰서 광고 + 제휴
주기     : 주 1회 발행
```

콘텐츠 소스 (무한):

| 소스 파일 | 콘텐츠 |
|---|---|
| `CHANGELOG.md` (262KB) | "Svelte 이번주 변경사항" 주간 연재 |
| `messages/` 폴더 | "이 에러 뜨면 이렇게" 트러블슈팅 시리즈 |
| `benchmarking/` | "성능 측정 실험실" 정기 코너 |
| `documentation/` | 공식문서 한글 해설 |

영문 병행 시 글로벌 독자 확보 가능 (Substack/Ghost).

---

### TIER 2 — 개발 필요, 수익성 높음

#### #4. AI UI 생성 SaaS ⭐⭐⭐⭐⭐ (가장 큰 기회)

```
상품     : "말하면 Svelte 컴포넌트가 나오는" 웹서비스
가격     : Free(월 10회) / Pro $19 / Team $99
스택     : SvelteKit + Claude API + svelte/compiler
MVP      : 1~2개월
```

아키텍처 (이 레포가 핵심 역할):

```
사용자: "대시보드 카드 컴포넌트 만들어줘"
   |
[Claude API] --> Svelte 코드 생성
   |
[svelte/compiler] compile() 로 즉시 검증
   |
   ├─ ❌ 에러/경고 --> 에러메시지를 AI에게 재전달해 자동 수정 (최대 3회)
   └─ ✅ 성공 --> 실시간 프리뷰 + 코드 다운로드
```

핵심 코드:

```js
import { compile } from 'svelte/compiler';

async function generateComponent(prompt, maxRetry = 3) {
  let feedback = '';
  for (let i = 0; i < maxRetry; i++) {
    const code = await callClaude(prompt + feedback);
    try {
      const { js, css, warnings } = compile(code, { generate: 'client' });
      if (warnings.length === 0) return { code, js, css };   // 성공
      feedback = `\n이전 시도의 경고를 수정해줘: ${warnings.map(w => w.message).join(', ')}`;
    } catch (e) {
      feedback = `\n이전 시도가 컴파일 실패했어: ${e.message}`;  // 자동 재시도
    }
  }
  throw new Error('생성 실패');
}
```

성립 근거:
- 경쟁 서비스(v0.dev 등)는 대부분 React 전용 → Svelte 시장 공백
- "컴파일 검증 루프" = 동작 보장, 경쟁사 대비 품질 우위
- Svelte 커뮤니티는 작지만 충성도 높음

주의: API 비용 관리 필수 (Free 티어 남용 방지, 프롬프트 캐싱 활용)

#### #5. 프리미엄 Svelte 컴포넌트/템플릿 판매 ⭐⭐⭐⭐

```
상품     : 대시보드/SaaS 스타터 킷
가격     : $49~$299 (1회 구매) or 번들 $499
판매처   : Gumroad / LemonSqueezy / 자체몰
제작기간  : 1~2개월/개
```

잘 팔리는 아이템:

| 상품 | 가격대 | 수요 |
|---|---|---|
| SaaS 어드민 대시보드 (SvelteKit+인증+결제) | $199~ | 🔥🔥🔥 |
| **AI 채팅 UI 킷** (스트리밍 최적화) | $99~ | 🔥🔥🔥 (현재 최고) |
| 데이터 테이블/차트 컴포넌트 | $79~ | 🔥🔥 |
| 랜딩페이지 템플릿 팩 | $49~ | 🔥🔥 |

React 템플릿 시장은 포화 + 가격경쟁. Svelte는 공급 부족 → 프리미엄 가격 유지 가능.

특별 추천: "AI 에이전트 채팅 UI 킷" (Svelte 5 스트리밍 성능 + AI 붐)

#### #6. 온라인 플레이그라운드 / 교육 플랫폼 ⭐⭐⭐

```
상품     : 브라우저에서 Svelte 실습 + 자동채점
수익원   : 구독 월 $9 / 기업 교육 라이선스
핵심     : svelte/compiler 를 브라우저(WASM/Worker)에서 실행
```

차별 포인트:

```
일반 플레이그라운드: 코드 쓰면 결과만 표시
차별화 버전       : "컴파일된 JS를 나란히 표시 + 왜 이렇게 되는지 설명"
```

기업 교육용 라이선스(사내 Svelte 온보딩)가 B2B 고단가.

---

### TIER 3 — 전문성 기반 고단가

#### #7. 기술 컨설팅 / 마이그레이션 대행 ⭐⭐⭐⭐

```
서비스   : Svelte 4 --> 5 마이그레이션, 성능 최적화 진단
단가     : 시급 10~20만원 / 프로젝트 500만~3000만원
필요조건 : 이 레포 깊이 이해 + 공개 포트폴리오
```

레포 자산 → 서비스 변환:

| 레포 자산 | 컨설팅 서비스 |
|---|---|
| `src/compiler/migrate/` | Svelte4→5 자동 마이그레이션 대행 |
| `benchmarking/compare/` | **"성능 진단 리포트"** 상품화 |
| `.agents/skills/performance-investigation` | 진단 프로세스 그대로 사용 |
| `tests/` 구조 | 고객사 테스트 체계 구축 자문 |

영업 전략:

```
1. sveltejs/svelte 에 PR 머지 (신뢰도 증명)
2. 유튜브 시리즈 B/C 로 전문성 공개
3. "무료 성능 진단 리포트" 1페이지 제공 --> 유료 전환
```

#### #8. AI 에이전트 워크플로우 컨설팅 ⭐⭐⭐⭐⭐ (가장 유망)

```
서비스   : "우리 회사 레포를 AI 친화적으로 개조"
단가     : 프로젝트 300만~2000만원 / 리테이너 월 200만원~
타겟     : AI 도입하려는데 방법을 모르는 개발팀 (= 대부분)
```

제공 내용 (전부 이 레포 패턴 기반):

```
- AGENTS.md / CLAUDE.md 설계 및 작성
- 업무별 SKILL.md 제작 (배포/테스트/리뷰/성능조사 등)
- 테스트 구조를 "AI가 검증 가능하게" 개선
- CI/autofix 파이프라인 구축
- MCP 서버 연동 (사내 DB/Jira/Slack)
- 팀 교육 워크샵
```

기회 요인:
- "AI 코딩 도입"은 많은 회사의 우선 과제인데 방법을 아는 사람이 적음
- **"Svelte 팀이 실제로 쓰는 패턴"** 이라는 레퍼런스 보유
- 수요 > 공급

상품화 래더:

```
무료   : 유튜브 "AGENTS.md 작성법" (유입)
저가   : SKILL.md 템플릿 팩 $49 (Gumroad)
중가   : 2시간 워크샵 50만원
고가   : 레포 개조 프로젝트 500만원~
최고가 : 월 리테이너 (지속 개선) 200만원/월
```

---

### 종합 비교표

| # | 아이디어 | 초기비용 | 수익규모 | 난이도 | 추천도 |
|:-:|---|:-:|:-:|:-:|:-:|
| 1 | 한글 유료강의 | 🟢 0 | 💰💰💰 | 🟡 중 | ⭐⭐⭐⭐⭐ |
| 2 | 유튜브 채널 | 🟢 0 | 💰💰 | 🟢 하 | ⭐⭐⭐⭐ |
| 3 | 블로그/뉴스레터 | 🟢 0 | 💰 | 🟢 하 | ⭐⭐⭐ |
| 4 | AI UI 생성 SaaS | 🟡 중 | 💰💰💰💰 | 🔴 상 | ⭐⭐⭐⭐⭐ |
| 5 | 컴포넌트/템플릿 판매 | 🟢 저 | 💰💰💰 | 🟡 중 | ⭐⭐⭐⭐ |
| 6 | 교육 플랫폼 | 🟡 중 | 💰💰 | 🔴 상 | ⭐⭐⭐ |
| 7 | Svelte 컨설팅 | 🟢 0 | 💰💰💰💰 | 🔴 상 | ⭐⭐⭐⭐ |
| 8 | **AI 워크플로우 컨설팅** | 🟢 0 | 💰💰💰💰💰 | 🟡 중 | ⭐⭐⭐⭐⭐ |

### 추천 실행 순서 (6개월 플랜)

```
1개월: 기반 다지기
  - 유튜브 시리즈 A(입문) 제작 시작
  - 블로그 주 1회 발행 (CHANGELOG 번역부터)
  - sveltejs/svelte 에 good first issue PR 도전

2~3개월: 신뢰 쌓기
  - 시리즈 C-EP5,6 (AI/AGENTS.md 소재) --> 바이럴 노림
  - 시리즈 B(내부해부) 제작 --> 전문성 증명
  - "AI 에이전트 채팅 UI 킷" 템플릿 1개 --> Gumroad

4~6개월: 수익 본격화
  - 시리즈 B 유료 강의 패키징 --> 인프런 등록
  - AI 워크플로우 컨설팅 랜딩페이지 (SvelteKit으로)
  - AI UI 생성 SaaS MVP 개발 착수
```

핵심 원칙: **TIER 1(콘텐츠)로 신뢰 → TIER 3(컨설팅)으로 고단가 → TIER 2(제품)로 확장.**
콘텐츠가 영업이고, 영업이 제품 검증이다.

> ⚠️ 면책: 위 수익 수치는 시장 가격대 기반 **추정치**이며 보장된 값이 아니다.

---

## 5. 부록 — 명령어 치트시트

### 개발

```bash
pnpm install                      # 의존성 설치
pnpm build                        # packages/* 빌드
pnpm check                        # 타입체크
pnpm lint                         # eslint + prettier 검사
pnpm format                       # prettier 자동 정리
```

### 테스트

```bash
pnpm test                         # 전체
pnpm test runtime-runes           # Svelte 5 반응성
pnpm test runtime-legacy          # Svelte 4 호환
pnpm test validator               # 검증 로직
pnpm test compiler-errors         # 에러 메시지
pnpm test server-side-rendering   # SSR
pnpm test hydration               # 하이드레이션
```

### 벤치마크 / 프로파일링

```bash
pnpm bench                                          # 전체
pnpm bench kairo_mux kairo_deep kairo_broad         # 선택 실행
pnpm bench:compare main my-branch                   # 브랜치 비교
pnpm bench:debug                                    # 디버거 연결
pnpm profile:compiler                               # 컴파일러 프로파일
node benchmarking/compare/profile-diff.mjs kairo_mux_owned main foo
```

### 샌드박스

```bash
cd playgrounds/sandbox && pnpm dev
```

### 릴리스 (메인테이너용)

```bash
npx changeset                     # 변경 기록 추가
pnpm changeset:version            # 버전 올리기
pnpm changeset:publish            # npm 배포
```

### 주요 파일 빠른 참조

| 알고 싶은 것 | 파일 |
|---|---|
| `$state` 구현 | `packages/svelte/src/internal/client/reactivity/sources.js` |
| `$derived` 구현 | `packages/svelte/src/internal/client/reactivity/deriveds.js` |
| `$effect` 구현 | `packages/svelte/src/internal/client/reactivity/effects.js` |
| 업데이트 스케줄링 | `packages/svelte/src/internal/client/reactivity/batch.js` |
| 객체 반응성 (Proxy) | `packages/svelte/src/internal/client/proxy.js` |
| 런타임 코어 | `packages/svelte/src/internal/client/runtime.js` |
| 파서 진입점 | `packages/svelte/src/compiler/phases/1-parse/index.js` |
| 분석 진입점 | `packages/svelte/src/compiler/phases/2-analyze/index.js` |
| 변환 진입점 | `packages/svelte/src/compiler/phases/3-transform/index.js` |
| 공개 API | `packages/svelte/src/compiler/index.js` |
| AI 에이전트 가이드 | `AGENTS.md` |
| AI 성능조사 스킬 | `.agents/skills/performance-investigation/SKILL.md` |
| 기여 가이드 | `CONTRIBUTING.md` |

---

## 6. 핵심 결론 3줄

1. 이 레포는 **Svelte 5 프레임워크의 전체 소스코드** 이며, 앱 개발용이 아니라 **Svelte 자체를 개발/학습하기 위한 것** 이다.
2. 가장 큰 숨은 가치는 `AGENTS.md` + `.agents/skills/SKILL.md` 구조 — **AI 에이전트에게 레포 작업법을 가르치는 업계 최상급 패턴 레퍼런스** 다.
3. MIT 라이선스 + 공식 문서 원본 + 컴파일러 API를 조합하면 **콘텐츠(강의/유튜브) → 컨설팅(AI 워크플로우) → 제품(AI UI 생성 SaaS)** 순서로 수익화 경로를 만들 수 있다.

---

*Generated by Claude Code — 카리나 💖*
*레포: https://github.com/bmshin94/svelte*
