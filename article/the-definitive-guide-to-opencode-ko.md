URL: https://blog.devgenius.io/the-definitive-guide-to-opencode-from-first-install-to-production-workflows-aae1e95855fb
번역 모델: Gemini 2.0 Flash

# OpenCode 결정판 가이드: 최초 설치부터 프로덕션 워크플로우까지

**JP Caparas 작성 | 2026년 1월 | Dev Genius**

8개월 만에 GitHub 스타 9만 개 이상을 획득하며 오픈소스 AI 코딩 에이전트의 정점에 선 OpenCode가 어떻게 여러분의 개발 워크플로우를 혁신할 수 있는지, 필자의 실제 프로젝트 예시와 함께 상세히 설명합니다.

---

OpenCode를 최대한 활용할 수 있도록 돕기 위해 이 가이드를 작성했습니다. 제 뉴스레터를 구독하시면 이런 소식을 정기적으로 받아보실 수 있습니다.

OpenCode의 성장은 경이로움 그 자체였습니다. 출시 후 1년도 채 되지 않아 9만 개 이상의 GitHub 스타를 모았는데, 이는 개발자 도구 역사상 가장 빠른 성장세 중 하나입니다.

심지어 Supabase의 창립자인 Paul Copplestone도 OpenCode에 깊은 인상을 받았습니다. OpenAI와 GitHub은 공식적으로 OpenCode와 파트너십을 맺었으며, Anthropic 계정 차단에 대한 우려도 이제 옛일이 되었습니다. Claude Code에 매달 20달러를 지불하던 개발자들은 이제 터미널에서 제약 없이 원하는 모델과 프로바이더를 사용할 수 있다는 사실을 깨닫고 있습니다.

저는 출시 직후부터 OpenCode를 매일 사용해 왔으며, 이를 활용해 멀티 에이전트 코드 리뷰 시스템을 구축하고 커밋 메시지부터 Cloudflare 배포까지 처리하는 커스텀 스킬을 만들었습니다. OpenCode가 초기 Claude Code의 대안 수준을 넘어, 이제는 모방하려 했던 대상보다 더 뛰어난 도구로 진화하는 과정을 지켜보았습니다.

이 가이드에는 제가 배운 모든 것을 담았습니다. 최초 설치부터 팀 동료들이 "도대체 생산성이 왜 이렇게 높아진 거야? 초사이어인이라도 된 건가?"라고 놀랄 만한 프로덕션 워크플로우까지 모두 다룹니다. 자, 시작해 봅시다.

## Part 1: 시작하기

### OpenCode란 정확히 무엇인가요?

OpenCode는 오픈소스 **AI 코딩 에이전트**입니다. 터미널, 데스크톱 또는 IDE에서 실행됩니다. 프로젝트의 코드베이스를 읽고 구조를 이해하며, 코드를 작성하고 명령어를 실행하고 사용자의 패턴을 학습합니다.

Claude Code, Cursor 또는 GitHub Copilot의 에이전트 기능을 사용해 보셨다면 개념은 익숙하실 겁니다. 차이점은 무엇일까요? OpenCode는 특정 프로바이더에 종속되지 않습니다. 기본적으로 75개 이상의 LLM 프로바이더를 지원합니다. Claude, GPT, Gemini는 물론 Kimi, DeepSeek, Qwen3 Coder와 같은 오픈소스 모델이나 Ollama를 통한 로컬 모델까지 사용할 수 있습니다. 인터페이스는 그대로 유지하면서 '두뇌'만 바꾸는 식이죠.

이것은 단순히 비용 절감의 문제가 아닙니다(물론 비용 절감 효과도 큽니다). 핵심은 **선택권**입니다. 새로운 모델이 출시되면 즉시 시도해 볼 수 있고, 더 저렴한 프로바이더가 나오면 워크플로우를 바꾸지 않고도 전환할 수 있습니다. 회사의 정책에 따라 특정 프로바이더를 사용해야 할 때도 도구를 포기할 필요가 없습니다.

### OpenCode가 이토록 빠르게 인기를 얻은 이유

2025년 중반에 세 가지 요인이 맞물렸습니다.

1.  **Anthropic 차단 위기:** Claude Code 사용자들이 Anthropic의 남용 탐지 시스템으로부터 이메일을 받기 시작했고, 계정이 정지되기도 했습니다. 비공식 통합을 통해 Copilot을 사용하던 워크플로우도 차단되면서 사용자들은 큰 불확실성에 직면했습니다.
2.  **프로바이더 자유 선언:** OpenCode는 단순한 대안이 아니라, 특정 프로바이더에 다시는 종속되지 않을 자유를 제안했습니다. 매달 AI 역량이 요동치는 업계에서 이는 일종의 보험과 같습니다.
3.  **SST의 혈통:** OpenCode는 서버리스 인프라 프레임워크로 유명한 SST 팀이 만들었습니다. 이들은 개발자 도구의 사용자층과 그들이 겪는 문제를 정확히 꿰뚫고 있었습니다.

### 사전 요구 사항

설치 전 두 가지가 필요합니다.

**1. 최신 터미널 에뮬레이터**
OpenCode의 TUI(터미널 사용자 인터페이스)는 트루 컬러(True Color)와 적절한 유니코드 지원이 필요합니다. 추천 도구는 다음과 같습니다.
*   **WezTerm** (멀티 플랫폼, 추천)
*   **Alacritty** (멀티 플랫폼, 매우 빠름)
*   **Ghostty** (Linux 및 macOS, 필자가 가장 선호함)
*   **Kitty** (Linux 및 macOS)
*   macOS 기본 Terminal.app도 작동은 하지만 시각적 효과가 떨어질 수 있습니다.

**2. LLM 프로바이더 API 키**
최소한 하나의 프로바이더 키가 필요합니다. 다음 중 하나로 시작하는 것을 추천합니다.
*   **Synthetic** (가장 쉬움, Kimi K2.5 등 강력한 오픈소스 모델 지원)
*   **OpenCode Zen** (엄선된 모델, 종량제)
*   **OpenAI** (ChatGPT Plus/Pro 구독 연동 가능)
*   **GitHub Copilot** (이제 공식 지원됨)

*참고: 필자는 Synthetic.new에서 제공하는 Kimi K2.5를 선호합니다. 사용자 데이터를 학습에 사용하지 않기 때문입니다.*

### 설치

가장 쉬운 방법:
`curl -fsSL https://opencode.ai/install | bash`

패키지 매니저를 사용하는 경우:
```bash
# macOS/Linux (Homebrew)
brew install anomalyco/tap/opencode

# Node.js
npm install -g opencode-ai

# Bun
bun install -g opencode-ai
```

Windows의 경우:
```bash
# Chocolatey
choco install opencode

# Scoop
scoop install opencode
```

설치 확인:
`opencode --version`

### 첫 실행: 프로바이더 연결

프로젝트 디렉토리로 이동하여 OpenCode를 실행합니다.
```bash
cd ~/projects/my-app
opencode
```
TUI가 로드되면 `/connect`를 입력하여 프로바이더를 연결합니다. 가장 일반적인 옵션은 다음과 같습니다.

*   **Synthetic (추천):** Synthetic에서 API 키를 발급받아 OpenCode에서 'Synthetic'을 선택하고 붙여넣습니다.
*   **OpenCode Zen:** 'opencode'를 선택하면 브라우저가 열립니다. 로그인 후 결제 정보를 등록하고 API 키를 복사해 터미널에 붙여넣습니다.
*   **GitHub Copilot:** 'GitHub Copilot'을 선택하고 브라우저 OAuth2 인증을 진행합니다. 이제 공식 지원되므로 차단 걱정 없이 사용할 수 있습니다.

### 프로젝트 초기화

프로바이더가 연결되었다면 프로젝트를 초기화합니다.
`/init`

이 명령은 코드베이스를 스캔하고 프로젝트 루트에 `AGENTS.md` 파일을 생성합니다. 이 파일은 AI를 위한 **프로젝트 지침서** 역할을 하며 다음과 같은 내용을 담습니다.
*   사용 중인 언어
*   프로젝트 구조
*   코딩 패턴 및 컨벤션
*   빌드 및 테스트 명령어

`AGENTS.md`가 구체적일수록 OpenCode의 성능이 향상됩니다. 저는 매주 이 파일을 업데이트하며 따랐으면 하는 패턴을 추가합니다.

## Part 2: 일상적인 워크플로우

### 두 가지 모드: 빌드(Build)와 플랜(Plan)

OpenCode에는 **Build**와 **Plan**이라는 두 가지 주요 에이전트가 있습니다. `Tab` 키로 전환할 수 있습니다.

*   🛠️ **Build 모드:** 모든 권한을 가집니다. 파일을 읽고, 코드를 쓰고, 명령어를 실행하며 변경 사항을 적용합니다. 기본 작업 모드입니다.
*   🗺️ **Plan 모드:** 권한이 제한됩니다. 읽기와 분석은 가능하지만 수정은 할 수 없습니다. 변경 전 제안을 받고 싶을 때 사용합니다.

**필자의 활용 패턴:**
1.  Plan 모드에서 기능을 설명하고 접근 방식의 개요를 요청합니다.
2.  계획을 검토하고 질문을 던져 구체화합니다.
3.  확신이 서면 Build 모드로 전환하여 "계획대로 구현해 줘"라고 요청합니다.

### 질문하기

OpenCode는 코드베이스를 이해하고 있습니다. 이를 활용하세요.
`@packages/api/src/auth/index.ts에서 인증은 어떻게 처리되고 있어?`

`@` 기호는 OpenCode가 특정 파일에 집중하게 합니다. 다음과 같은 포괄적인 질문도 가능합니다.
*   `이 프로젝트에서 에러 로그는 어디에 기록돼?`
*   `API 라우트 구조를 보여줘.`
*   `현재 존재하는 데이터베이스 마이그레이션은 뭐야?`

### 변경 사항 적용

간단한 변경은 바로 설명하면 됩니다.
`GET /health에서 { status: "ok" }를 반환하는 헬스 체크 엔드포인트를 추가해 줘.`

복잡한 기능은 Plan 모드를 먼저 활용하세요.
1.  [Tab으로 Plan 모드 이동]
2.  "노트에 소프트 딜리트(soft-delete) 기능을 추가하고 싶어. 삭제 플래그를 만들고, 최근 삭제된 항목 화면을 만들고, 복구 기능을 넣으려고 해. 어떻게 접근하면 좋을까?"
3.  계획 검토 후 [Tab으로 Build 모드 이동] "좋아, 진행해."

### 이미지 사용

이미지를 터미널로 드래그 앤 드롭할 수 있습니다. OpenCode는 이미지를 스캔하여 컨텍스트에 포함합니다.
*   **UI 목업:** "이 이미지처럼 보이는 컴포넌트를 만들어 줘."
*   **에러 스크린샷:** "이런 에러가 발생하는데 원인이 뭐야?"
*   **다이어그램:** "여기 그려진 아키텍처를 구현해 줘."

### 실행 취소(Undo) 및 다시 실행(Redo)

실수했나요? `/undo`를 사용해 마지막 응답의 변경 사항을 되돌릴 수 있습니다. 다시 적용하려면 `/redo`를 사용합니다.

### 대화 공유

`/share` 명령어는 현재 대화의 공유 링크를 생성합니다. 팀원에게 도움을 요청하거나 문서화할 때 유용합니다.

### 세션 재개

OpenCode를 닫았다가 다시 시작하고 싶다면 `/sessions`(또는 `/resume`, `/continue`)를 입력합니다.
단축키: `Ctrl + X, L`

명령줄에서 직접 마지막 세션을 이어갈 수도 있습니다.
`opencode --continue` 또는 `opencode -c`

### 세션 이름 지정

나중에 세션을 쉽게 찾으려면 이름을 지정하는 것이 좋습니다.
`opencode run --title "결제 API 리팩토링" "결제 처리 모듈을 Stripe를 사용하도록 리팩토링해 줘"`
이름이 지정된 세션은 나중에 `/sessions`로 찾기가 훨씬 수월합니다.

## Part 3: 구성(Configuration) 심층 분석

### `opencode.json` 파일

설정은 프로젝트 루트의 `opencode.json` 파일에서 관리합니다.
```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "synthetic/hf:moonshotai/Kimi-K2.5",
  "theme": "opencode",
  "autoupdate": true
}
```
`$schema`를 추가하면 JSON 스키마를 지원하는 에디터에서 자동 완성을 사용할 수 있습니다. (필자는 주석을 달기 위해 `.jsonc` 확장자를 사용하기도 합니다.)

### 설정 위치 및 우선순위(Precedence)

OpenCode는 다음 순서대로 설정을 로드하며, 나중에 로드된 설정이 이전 설정을 덮어씁니다.
1.  **원격 설정** (`.well-known/opencode`): 조직 차원의 기본값
2.  **글로벌 설정** (`~/.config/opencode/opencode.json`): 개인별 선호도
3.  **프로젝트 설정** (`opencode.json`): 프로젝트별 특정 설정

### 모델 설정

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "synthetic/hf:moonshotai/Kimi-K2.5",
  "small_model": "synthetic/hf:MiniMaxAI/MiniMax-M2.1"
}
```
`model`은 주력으로 사용할 모델이며, `small_model`은 세션 제목 생성과 같은 가벼운 작업을 처리합니다.

### 권한 설정 (Permissions)

OpenCode의 작업을 제어하고 싶다면 다음과 같이 설정합니다.
```json
{
  "permission": {
    "edit": "ask",
    "bash": "ask"
  }
}
```
*   `"allow"`: 승인 없이 실행
*   `"ask"`: 실행 전 매번 확인
*   `"deny"`: 실행 차단

### 포맷터 (Formatters)

OpenCode는 코드 작성 후 자동으로 포맷팅을 수행할 수 있습니다. Prettier나 Biome을 지원합니다.
```json
{
  "formatter": {
    "prettier": {
      "extensions": [".js", ".ts", ".jsx", ".tsx", ".json"]
    }
  }
}
```

### 커스텀 지침 (Custom Instructions)

`AGENTS.md` 외에도 추가 지침 파일을 지정할 수 있습니다.
```json
{
  "instructions": [
    "CONTRIBUTING.md",
    "docs/coding-guidelines.md",
    ".cursor/rules/*.md"
  ]
}
```

## Part 4: 프로바이더 (Providers)

OpenCode는 75개 이상의 LLM 프로바이더를 지원합니다.

### Synthetic (필자가 가장 선호하는 프로바이더)

Synthetic은 토큰당 과금이 아닌 **정액 요금제(Subscription)**를 제공하여 비용 걱정 없이 사용할 수 있습니다. Moonshot AI의 **Kimi K2.5** 모델은 1조 개의 파라미터를 가진 MoE(Mixture-of-Experts) 모델로, 코딩 작업에서 놀라운 성능을 보여줍니다.
*   **장점:** 토큰 소모에 대한 불안감 해소, Claude 대비 3~6배 높은 속도 제한, 에이전트 중심 워크플로우에 최적화.
*   **요금제:** Standard($20/월), Pro($60/월).

### OpenCode Zen (공식 큐레이션)

OpenCode 팀이 직접 운영하는 게이트웨이입니다.
*   **장점:** 설정이 간편함, 무료 모델(Kimi K2.5 Free 등) 제공, 팀 관리 기능.

### GitHub Copilot

이제 공식적으로 지원되므로 기존 Copilot 구독을 통해 GPT-5나 Claude 모델을 OpenCode에서 사용할 수 있습니다.

## Part 5: 확장성 3요소 (커맨드, 스킬, 에이전트)

OpenCode의 핵심 생산성은 커맨드, 스킬, 에이전트에서 나옵니다.
*   **커맨드(Commands):** '무엇(What)'을 할 것인가. 반복적인 프롬프트를 `/` 명령어로 자동화합니다.
*   **스킬(Skills):** '어떻게(How)' 할 것인가. 특정 작업을 수행하는 방법론을 학습시킵니다.
*   **에이전트(Agents):** '누가(Who)' 할 것인가. 특정 역할(리뷰어, 보안 전문가 등)을 수행하는 페르소나를 정의합니다.

### 커맨드: 반복 작업의 단축키

`.opencode/commands/test.md` 예시:
```markdown
---
description: 커버리지와 함께 테스트 실행
agent: build
---
테스트 스위트를 실행하고 커버리지 보고서를 생성해 줘.
실패한 테스트에 집중하고 해결책을 제안해 줘.
```
이제 터미널에서 `/test`만 입력하면 됩니다.

### 스킬: 재사용 가능한 지식

스킬은 에이전트가 필요할 때마다 참조하는 지침 파일입니다. `.opencode/skills/git-release/SKILL.md`에 릴리스 프로세스를 정의해 두면, 에이전트가 릴리스 준비 시 자동으로 이 스킬을 로드하여 일관된 작업을 수행합니다.

### 에이전트: 특화된 페르소나

에이전트는 특정 목적을 위해 구성된 AI 어시스턴트입니다. 모델, 권한, 도구 접근 권한을 다르게 설정할 수 있습니다.
*   **Build:** 기본 작업 모드 (모든 권한)
*   **Plan:** 분석 및 계획 모드 (읽기 전용)
*   **Reviewer (커스텀):** 코드 리뷰에 특화된 에이전트

## Part 6: 플러그인과 자동화

OpenCode의 플러그인 시스템은 TypeScript로 작성됩니다. 이를 통해 에이전트의 생명주기에 개입할 수 있습니다.
예를 들어, 에이전트가 코드를 수정한 직후 자동으로 린트(Lint)를 실행하고 에러가 있다면 다시 수정하게 하는 플러그인을 만들 수 있습니다.

## Part 7: MCP 서버

**MCP (Model Context Protocol)**를 사용하면 외부 도구와 데이터 소스를 OpenCode에 연결할 수 있습니다. 웹 검색, 데이터베이스 쿼리, PDF 처리 등을 가능하게 해줍니다.

## Part 8: 프로덕션 워크플로우

실제 업무에 적용 가능한 몇 가지 워크플로우 예시를 소개합니다.

### 워크플로우: 멀티 에이전트 코드 리뷰

단 한 번의 명령으로 세 명의 전문가 에이전트가 서로 다른 관점에서 PR을 분석하도록 할 수 있습니다.
*   **Review-Frontend:** React 패턴, 접근성, 성능(렌더 사이클) 집중 리뷰
*   **Review-Backend:** API 설계, 데이터베이스 쿼리(N+1 문제), 보안 리뷰
*   **Review-Infra:** 배포 설정, 환경 변수, 모니터링 리뷰

오케스트레이터 커맨드(`/review`)를 통해 이들의 분석 결과를 하나의 리포트로 종합합니다.

### 워크플로우: 기능 구현 파이프라인

복잡한 기능을 구현할 때 분석 -> 계획 -> 구현 -> 검증의 4단계 파이프라인을 정의하여 에이전트가 체계적으로 작업하도록 합니다.

### 워크플로우: 자동화된 PR 준비

`git diff`와 `git log`를 분석하여 PR 요약, 테스트 노트, 마이그레이션 단계를 포함한 설명을 자동으로 작성하고 `gh pr create` 명령어를 출력합니다.

## Part 9: 6개월 사용 후기 팁

*   **구체적인 제약 조건 명시:** "signup 폼에 유효성 검사를 추가해 줘. Zod를 사용하고 인라인 에러 메시지를 보여줘."처럼 구체적으로 지시하세요.
*   **컨텍스트 관리:** 세션이 너무 길어져 답변이 느려지면 `Ctrl + N`으로 새 세션을 시작하세요.
*   **비용 관리:** 복잡한 작업에는 고성능 모델을, 간단한 질문에는 저렴한 모델을 사용하도록 에이전트별로 모델을 다르게 설정하세요.

## Part 10: 향후 전망

OpenCode는 매우 빠르게 발전하고 있습니다.
*   **데스크톱 앱 및 IDE 확장 기능:** 터미널 외에도 익숙한 IDE 환경으로 통합이 확장될 예정입니다.
*   **팀 워크스페이스:** 조직 내 권한 관리 및 비용 한도 설정 기능이 강화됩니다.
*   **플러그인 생태계:** 커뮤니티 플러그인이 활성화되면서 특정 프레임워크에 특화된 도구들이 늘어날 것입니다.

### 시작하기 체크리스트

1.  OpenCode 설치 (5분)
2.  프로바이더 연결 (2분)
3.  `/init`으로 프로젝트 초기화 (1분)
4.  코드베이스에 대해 질문해 보기
5.  자주 하는 작업을 위한 커스텀 커맨드 만들기

학습 곡선은 완만하지만, 생산성 향상은 비약적입니다. 지금 바로 시작해 보세요!

---

**관련 참고 자료:**
*   [OpenCode 공식 문서](https://opencode.ai)
*   [OpenCode GitHub 저장소](https://github.com/anomalyco/opencode)
*   [OpenCode Zen](https://opencode.ai/auth)

가이드에 대해 궁금한 점이 있다면 Twitter(@zenoware)나 LinkedIn(@jpcaparas)으로 문의해 주세요.
