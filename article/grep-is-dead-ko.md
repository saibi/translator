> 원문 출처: https://x.com/artemxtech/status/2028330693659332615
> 원문 기사: https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things
> 번역 CLI: claude

# Grep는 죽었다: Claude Code가 실제로 기억하게 만든 방법

**Claude Code와의 모든 대화는 처음부터 시작됩니다. 로컬 검색 엔진과 한 마디도 타이핑하기 전에 전체 컨텍스트를 불러오는 스킬로 이 문제를 해결한 방법을 소개합니다.**

Claude Code와의 모든 대화는 처음부터 시작됩니다. 나는 3주 동안 700개의 세션을 진행했는데, 예전에 무슨 작업을 하고 있었는지 기억이 나지 않습니다. 무슨 일이 일어나고 있는지 통제력을 잃어가고 있습니다.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/09-session-graph.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/09-session-graph.png)
*일주일간의 작업을 시각화한 그래프. 각 노드는 세션이고, 각 연결선은 해당 세션이 건드린 파일입니다.*

새 터미널을 열면 상황 파악이 막막합니다. 프로젝트가 뭔지, 어떤 결정들을 내렸는지, 모든 컨텍스트를 어떻게든 찾아야 합니다. 매번 처음부터 다시 시작해야 하죠.

세션 도중에 상황은 더 나빠집니다. 컨텍스트가 60% 한도에 도달하면 압축하거나 핸드오프해야 합니다. 그러면 결정의 절반이 사라져 버립니다. 심지어 다음 날 계속 작업하려고 하면, 예전에 무슨 일이 있었는지 기억조차 나지 않습니다.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-session-graph-full.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-session-graph-full.png)
*확대 보기: 보라색 노드는 각각의 대화이고, 색깔 있는 사각형은 생성하거나 수정한 파일입니다. 스킬, 레시피, 콘텐츠 초안 — 모든 것이 뒤엉켜 있습니다.*

파일을 grep으로 뒤지는 현재 패러다임은 확장되지 않습니다. 그래서 내 볼트(vault)에 [QMD](https://github.com/tobi/qmd)를 연결했습니다. QMD는 Shopify CEO인 Tobias Lütke가 만든 검색 엔진입니다. 캐나다산입니다.

지금부터 소개할 전체 메모리 시스템은 하나의 스킬입니다. [2분 안에 설치](https://memory-artemzhutov.netlify.app/)하면 Claude Code가 이미 사용 방법을 알고 있습니다.

## QMD: 볼트를 위한 로컬 검색 엔진

QMD는 지식 베이스(knowledge base)를 위한 로컬 검색 엔진입니다. Obsidian 볼트를 인덱싱하고 1초 이내에 무엇이든 찾아줍니다.

각 볼트 폴더마다 QMD 컬렉션(collection)이 있습니다. 노트, 일일 기록, 세션, 트랜스크립트. 일대일 매핑입니다. 각 컬렉션에서 집중적인 검색을 수행할 수 있습니다.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-collections-concept.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-collections-concept.png)

```bash
qmd collection list
qmd search "video workflow" -c notes -n 3
```

끝입니다. 명령어 하나로 볼트를 검색할 수 있습니다.

그런데 어떤 방식으로 검색할까요? Claude Code의 기본 검색 방식은 무차별 대입(brute force)입니다 — Haiku 서브에이전트(sub-agent)를 보내 모든 파일을 grep으로 뒤집니다. 직접 시도해봤는데([영상 보기](https://youtu.be/RDoTY4_xh0s?t=167)), 다양한 그래프 접근 방식에 대한 노트를 일반적인 방법으로 검색했더니 3분이나 걸렸습니다. 지루해서 Twitter를 스크롤하며 기다렸습니다. 결과는? 파일 300개, 별로였습니다.

QMD 검색은 즉각적이었습니다. 더 나은 결과, 훨씬 적은 토큰. 서브에이전트를 사용할 필요가 없습니다. 차이점은 이렇습니다: grep은 문자열을 매칭하지만, QMD는 관련성으로 순위를 매깁니다.

## Grep vs BM25 vs 시맨틱 검색 ([영상 보기](https://youtu.be/RDoTY4_xh0s?t=1656))

QMD는 세 가지 검색 모드를 제공합니다. **BM25** (`qmd search`)는 결정론적(deterministic) 전문 검색(full-text search)입니다. grep처럼 정확한 키워드를 매칭하지만, grep과 달리 각 파일에 점수를 매깁니다: 단어가 얼마나 자주 나타나는지, 그리고 모든 문서에서 얼마나 희귀한지? "sleep"을 다섯 번 언급하는 짧은 노트는 "sleep"이 한 번 등장하는 1만 자 파일보다 높은 점수를 받습니다. AI도, 임베딩도 없습니다 — 순수한 수학입니다. **시맨틱 검색** (`qmd vsearch`)은 임베딩을 사용해 의미를 찾습니다 — 정확한 단어가 없어도 개념으로 검색할 수 있습니다. **하이브리드** (`qmd query`)는 둘 다 결합합니다.

볼트에서 "sleep"을 검색해봤습니다. 차이를 확인해보세요.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-benchmark-results.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-benchmark-results.png)
*벤치마크: grep은 사방에서 200개의 노이즈 파일을 반환합니다. BM25는 2초 만에 관련 수면 실험을 찾습니다. 시맨틱 검색은 정확한 키워드 없이도 의미를 찾아냅니다.*

**Grep**: 200개 파일을 찾았습니다. 사방팔방입니다. "sleep"이라는 단어가 들어있는 모든 파일을 찾습니다. 심지어 코드 실행을 일시 중지하는 프로그래밍 명령어인 `sleep()`도 찾습니다. 실제 수면과는 전혀 관련이 없습니다. 이것이 문자열 매칭의 문제입니다.

**BM25** (2초): 수면 질에 대한 회고, 수면 단편화 패턴 추적 실험, 새벽 3시에 수면 방해. 훨씬 낫습니다.

하지만 `qmd search "insomnia"`는 결과가 0개입니다. 볼트에 그 단어가 존재하지 않기 때문입니다.

**시맨틱 검색**: `qmd vsearch "couldn't sleep, bad night"`을 입력하니 몇 년 전에 세웠던 취침 시간 규율 목표가 나왔습니다. 키워드를 넘어 의미를 탐구합니다. 결과의 네 개 중 다섯은 검색어를 전혀 포함하지 않습니다. Grep은 절대 찾을 수 없었을 것입니다.

**하이브리드** (`qmd query "couldn't sleep, bad night"`): 수면 질 개선 89%, 새벽 3시 수면 방해 51%, 건강 수면 최적화 42%. 가장 좋은 순위 매기기입니다.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-grep-bm25-semantic.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-grep-bm25-semantic.png)
*발전 과정: grep은 문자열을 잡고, BM25는 관련성을 잡고, 시맨틱 검색은 의미를 잡습니다.*

BM25부터 시작하세요. 빠르고 구조화된 노트를 잘 처리합니다. 검색의 80%를 커버합니다. 정확한 단어로 절대 검색하지 않을 트랜스크립트와 브레인덤프(braindump)에는 시맨틱 검색을 추가하세요.

## 시맨틱 검색이 실제로 찾아내는 것 ([영상 보기](https://youtu.be/RDoTY4_xh0s?t=439))

일일 노트를 검색했습니다: "내가 행복했던 날과 그 이유를 찾아줘."

이것은 간단한 쿼리가 아닙니다. Claude가 이를 조정해 여러 검색을 실행했습니다:

```bash
qmd vsearch "happy, grateful, excited" -c daily -n 5
qmd vsearch "energy, great day, feeling good" -c daily -n 5
qmd vsearch "satisfaction, accomplishment" -c daily -n 5
```

수개월에 걸친 일일 노트에서 의미적으로 관련 있는 연결들을 찾아냈습니다.

패턴: 내가 가장 행복한 날들은 무언가를 출시(ship)했고 사우나나 9시간 수면 같은 좋은 수면 회복이 있었을 때입니다.

그런 다음 완전히 잊고 있던 것이 드러났습니다. 10월, 박사 논문을 쓰면서 포기 직전이었을 때. 매일 나와서 뭔가를 써야 했습니다. 하지만 그때 깨달은 것은 빠른 해결책으로 도망치는 대신 불편함을 직면해야 한다는 것이었습니다.

그 글을 썼다는 사실을 기억하지 못했습니다. 검색이 이걸 찾아낼 거라고 기대하지도 않았습니다. 그런데 거기 있었습니다, 정확한 인용문이.

## /recall - 시작 전에 컨텍스트 불러오기 ([영상 보기](https://youtu.be/RDoTY4_xh0s?t=813))

`/recall`은 QMD 위에서 동작하는 Claude Code 스킬입니다. 작업을 시작하기 전에 컨텍스트를 불러옵니다 — Claude에게 무엇을 하고 있었는지 설명하는 대신, recall하라고 지시하면 됩니다.

세 가지 모드가 있습니다: 시간적(temporal, 날짜별로 세션 기록 스캔), 주제별(topic, QMD 컬렉션 전체 BM25 검색), 그래프(graph, 세션과 파일의 인터랙티브 시각화).

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-recall-three-modes.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-recall-three-modes.png)
*세 가지 모드: 시간적(어제, 지난 주), 주제별(컬렉션 전체 BM25), 그래프(시각적 탐색).*

```
/recall yesterday
/recall topic graph
/recall graph last week
```

**Yesterday** 모드는 하루 39개 세션을 재구성했습니다. 타임라인, 각 세션의 메시지 수, 언제 무엇을 작업했는지.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-recall-yesterday-timeline.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-recall-yesterday-timeline.png)
*하루 39개 세션. 각 행은 시간, 메시지 수, 작업 내용을 보여줍니다. 그 중 어느 것으로든 돌아갈 수 있습니다.*

**Topic** 모드는 세션과 노트 전체에서 "QMD video"를 검색했습니다. 대시보드, 제작 계획, 할 일 목록이 반환됐습니다. 관련 파일이 1분도 안 돼 모두 나왔습니다. 무차별 대입 접근법과 비교해보세요: Claude에게 "이 프로젝트에 대한 모든 정보를 찾아줘"라고 하면 Haiku를 보내 3분 동안 볼트를 grep으로 뒤지고, 토큰을 소비하고, 더 나쁜 결과를 가져옵니다. `/recall topic`으로는 프로젝트의 전체 상태를 재구성하고 물었습니다: 지금 가장 레버리지가 높은 다음 행동은 무엇인가요?

**Graph** 모드는 내 전체 한 주의 인터랙티브 시각화를 열었습니다. 컬러 블롭(blob)으로 된 세션들, 오래된 것은 더 흐릿하게, 최근 것은 보라색으로 강조됩니다. 파일은 유형별로 클러스터링됩니다: 목표, 리서치, 음성, 문서, 콘텐츠, 스킬.

예를 들면 이렇습니다: 점심 장소를 탐색하고 있었습니다. Claude에게 "좋은 점심 먹고 싶어"라고 했고 여러 장소를 분석했습니다. 해볼 만한 활동들로 저장해뒀습니다. 일주일 후 그래프를 열고, 그 세션을 발견하고, 해당 파일 경로를 Claude Code에 복사해 거기서 이어서 작업합니다. 그래프는 모든 과거 대화를 복구 가능하게 만듭니다.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-session-graph-tree.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-session-graph-tree.png)
*내 한 주의 활동 그래프. 세션이 파일들로 분기됩니다. 어떤 대화든 그것이 만들어낸 것으로 추적할 수 있습니다.*

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-session-graph-legend.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-session-graph-legend.png)
*세션 그래프 컨트롤: 시간 범위별 필터, 파일 유형별 색상 코딩. 오래된 세션은 어두운 보라색으로 희미해지고, 최근 것은 밝게 빛납니다. 호버하면 연결이 강조되고, 클릭하면 선택됩니다.*

## 700개 세션, 모두 검색 가능 ([영상 보기](https://youtu.be/RDoTY4_xh0s?t=2226))

Claude Code는 모든 대화를 컴퓨터에 JSONL 파일로 저장합니다. 지난 3주 동안 700개의 세션이 있었습니다. 각각에는 결정, 질문, 다시 필요할 컨텍스트가 담겨 있습니다.

원시 파일에는 tool use, 시스템 프롬프트, 역할(role)이 있습니다. 우리는 명확한 마크다운으로 파싱합니다 — 실제 신호, 실제 사용자 메시지 — 그리고 이것을 QMD 인덱스에 임베드합니다.

각 세션이 끝나고 터미널을 닫을 때, 훅(hook)이 실행되어 그 세션들을 QMD에 익스포트하고 임베드합니다. 그래서 인덱스는 항상 최신 상태입니다. 오늘 작업한 것도 다시 찾아볼 수 있습니다.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-session-pipeline.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-session-pipeline.png)
*세션이 닫히면, 훅이 발동되고, QMD 인덱스가 업데이트됩니다. 항상 최신, 수동 작업 없음.*

## 실행하지 않은 아이디어 찾기 ([영상 보기](https://youtu.be/RDoTY4_xh0s?t=2078))

이건 저도 놀랐습니다. 이렇게 검색했습니다: "내가 한 번도 실행하지 않은 아이디어를 찾아줘."

QMD 원시 결과를 Claude가 합성해주는 것이 매우 유용합니다 — 원시 결과를 직접 파싱하기는 어렵습니다. Claude가 찾은 것을 요약해줬습니다:

- 10월 19일 — 박사 논문 작성 대시보드를 만들고 싶었지만 한 번도 하지 않았음
- 일러스트 기반 앱 아이디어가 있었지만 실행하지 않았음
- 내 전체 Obsidian 워크플로우에 대한 화면 녹화를 녹화하려 했지만 실행에 옮기지 않았음

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-ideas-never-acted-on.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-ideas-never-acted-on.png)
*하이브리드 검색 결과에 대한 Claude의 합성. 날짜, 아이디어, 수개월 전에 적고 완전히 잊어버린 것들.*

그리고 이 모든 것은 로컬입니다 — 모든 임베딩이 내 컴퓨터에 저장됩니다.

## 도구는 바뀝니다. 당신의 컨텍스트는 남습니다.

노트는 더 이상 수동적이지 않습니다. Obsidian 세계에 갇혀 있지 않습니다. 실제로 무언가를 하기 시작합니다. 당신의 모든 노트는 삶에서 목표를 달성하는 데 사용할 수 있는, 자신에 관한 유용한 컨텍스트입니다.

도구는 바뀝니다. 한 달 후에는 새 모델들이 등장할 것입니다. 그래도 상관없습니다. 컨텍스트가 있다면 어떤 상황에서도 작동하게 만들 수 있습니다. Claude Code, Codex, Gemini CLI, 무엇이든.

메모리 레이어(memory layer)는 전체 스택(stack)에 걸쳐 스킬로 작동합니다. Obsidian Sync를 사용해 Mac과 항상 켜져 있는 Mac Mini 사이에 볼트를 동기화합니다. Mac Mini에서는 OpenClaw가 24/7 실행됩니다. 그래서 휴대폰을 들고, OpenClaw를 열면, 어디서든 같은 볼트, 같은 QMD 인덱스, 같은 스킬을 사용할 수 있습니다.

![](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-full-stack.png)
[원본 이미지](https://artemxtech.github.io/Grep-Is-Dead-How-I-Made-Claude-Code-Actually-Remember-Things/assets/qmd-full-stack.png)
*전체 스택: 하단에 Obsidian 볼트, 중간에 QMD 검색, 상단에 Claude Code와 OpenClaw. 컨텍스트가 위로 흐릅니다.*

## 오늘 시작하기

`/recall` 스킬을 다운로드하고, `.claude/skills/` 폴더에 넣으면 오늘 바로 세션 파이프라인과 recall이 작동합니다 (또는 Claude에게 대신 해달라고 요청하세요 :) ).

전체 시스템을 원한다면, **[Claude Code x Obsidian Lab](https://lab.artemzhutov.com/)** 을 운영하고 있습니다 — 6주, 실전 중심. 처음부터 시작합니다: Obsidian 설정, Claude Code 설정, 모두가 속도를 낼 수 있도록. 그런 다음 제가 보여준 것 이상으로 나아갑니다. 자신만의 스킬, 자신만의 워크플로우, 실제로 컨텍스트를 아는 자신만의 에이전트를 구축합니다. 격주 워크숍과 마스터마인드에서 그룹으로부터 실질적인 피드백을 받습니다. 모두가 비슷한 문제를 겪고 있습니다 — 정보 과부하, Claude가 만드는 혼란, Obsidian 튜닝에 실제 작업보다 더 많은 시간을 쓰는 것. 코호트(cohort) 2는 3월 17일에 시작합니다.

## 함께 논의해요

Claude Code 메모리에서 가장 큰 고통은 무엇인가요? 매 세션마다 컨텍스트를 다시 설명하고 있나요?

Discord 커뮤니티에 참여하세요: [https://discord.gg/g5Z4Wk2fDk](https://discord.gg/g5Z4Wk2fDk)

## 리소스

**QMD** — 이 모든 것의 기반이 되는 검색 엔진. Tobias Lütke 제작.
[https://github.com/tobi/qmd](https://github.com/tobi/qmd)

**Claude 메모리 시스템** — /recall — 다운로드하고 오늘 바로 시작.
[https://memory-artemzhutov.netlify.app](https://memory-artemzhutov.netlify.app)
가장 빠른 시작 방법: Claude Code에게 레포에 포함된 설정 가이드를 읽고 모든 것을 설치해달라고 하세요.

**Claude Code x Obsidian Lab** — 노트를 사용하는 AI 워크플로우 구축 6주 실전 과정. 코호트 2는 3월 17일 시작. 얼리버드 가격은 이번 주에 종료됩니다.
[https://lab.artemzhutov.com](https://lab.artemzhutov.com)

**전체 영상 보기** — 라이브 데모가 포함된 42분 워크스루(walkthrough).
[https://youtu.be/RDoTY4_xh0s](https://youtu.be/RDoTY4_xh0s)

Artem

---

> 번역 CLI: claude
