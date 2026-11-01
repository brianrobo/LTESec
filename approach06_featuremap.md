맞아요. 그런데 정확히는 **12:18 자체에서는 feature map 설명이 거의 끝나고, 그 결과로 “Agent가 자기 일을 검증할 수 있게 되었다”는 결론으로 넘어가는 시점**입니다. Feature map 설명은 그 직전 구간에 나옵니다. [Prepublish](https://prepublish.ai/youtube-transcript/NjoZoUm85x0?utm_source=chatgpt.com)

Lauren이 말하는 **Feature Map**은 쉽게 말하면:

> **Agent가 “이 제품에 어떤 기능이 있고, 사용자가 그 기능에 어떻게 도달하며, 자동화 도구로는 어떻게 조작해야 하는지” 알 수 있게 해주는 제품 지도**

입니다.

처음에 Lauren은 `Control Glass`를 만들어 Agent가 Cursor 앱을 실행하고 trace도 뜨고 UI도 조작할 수 있게 했습니다. 그런데 문제가 있었습니다.

```text
Control Glass 있음
→ 앱 실행 가능
→ 클릭 가능
→ trace 가능

그런데...

Agent:
"왼쪽 sidebar가 어디지?"
"PR tab은 어떻게 열지?"
"사용자가 말한 이 화면이 어느 feature지?"
```

즉 **손과 눈은 생겼는데, 제품에 대한 지도가 없었던 것**입니다. [태호](https://taeho.io/en/reading/cursor-s-lauren-tan-on-trusting-and-verifying-ai-agents_26300?utm_source=chatgpt.com)

Lauren이 든 예가 아주 명확합니다.

사용자가 Slack에:

```text
"left sidebar가 느려요"

또는

"PR tab이 안 돼요"

또는

스크린샷 하나 + "???"
```

정도만 보내면, Agent는 앱을 실행할 능력은 있지만 **그 기능이 어디 있는지, 어떻게 접근하는지 몰라 헤맵니다.** 코드도 뒤지고 UI도 여기저기 클릭하면서 시간을 낭비합니다. [Prepublish](https://prepublish.ai/youtube-transcript/NjoZoUm85x0?utm_source=chatgpt.com)

그래서 Feature Map을 만든 겁니다.

예를 들면 개념적으로 README 안에 이런 정보가 있는 셈입니다.

```markdown
# Feature Map

## Agent Sidebar

User location:
- Left side of Agent Window

How to reach:
- Open Agent Window
- Click sidebar button
- Shortcut: ...

Sub-features:
- Chat list
- New chat
- Search

Automation:
- DOM selector: ...
- CDP attribute: ...

Related code:
- ...
```

즉 feature map은 단순한 기능 목록이 아닙니다.

```text
Feature 이름
+
사용자 관점 위치
+
접근 방법
+
keyboard shortcut
+
DOM selector / attribute
+
sub-feature
+
기능 설명
```

까지 들어갑니다. Lauren은 이를 **site map에서 아이디어를 얻은 “materialized memory”**라고 설명합니다. 즉 “이 앱이 어떻게 생겼고 어떻게 사용하는지”를 Agent가 매번 추론하지 않도록 문서로 외부화해놓은 겁니다. [Prepublish](https://prepublish.ai/youtube-transcript/NjoZoUm85x0?utm_source=chatgpt.com)

그래서 앞에서 본 CLI와 Feature Map의 역할이 서로 다릅니다.

```text
CLI / Control Glass
= 어떻게 조작할 것인가

Feature Map
= 무엇을 어디에서 조작해야 하는가
```

둘을 합치면:

```text
User report
"PR tab이 이상함"
      ↓
Feature Map
"PR tab은 여기 있고,
이렇게 접근한다"
      ↓
CLI / Playwright / CDP
실제로 앱 이동
      ↓
기능 재현
      ↓
Trace / Screenshot / Console
      ↓
Verification
```

이 구조가 됩니다. [태호](https://taeho.io/en/reading/cursor-s-lauren-tan-on-trusting-and-verifying-ai-agents_26300?utm_source=chatgpt.com)

그리고 이걸 지금 우리가 얘기하는 **Modem Domain Verification**에 옮기면 꽤 재미있습니다.

예를 들어 륜님이 NAS mobility/security domain을 대상으로 한다면, modem 쪽 Feature Map에 해당하는 건 이런 겁니다.

```text
Domain Map

Feature:
RAT Downgrade Detection

Entry:
- NAS Reject indication
- RAT change indication
- Cell information

Required Context:
- Current RAT
- Previous RAT
- Security context
- Reject cause
- Cell ID

Relevant Modules:
- nas_xxx.c
- mobility_xxx.c
- security_xxx.c

Flow:
Reject Event
→ Mobility Decision
→ RAT Change
→ Security Evaluation

Boundary Output:
- Network Security Event
- Category
- Reason Code
```

즉 modem에서는 굳이 이름을 `Feature Map`이라고 하지 않더라도,

> **특정 protocol feature에 대해 Entry / Context / Flow / Output / Relevant Module을 정리한 Domain Map**

을 만들 수 있습니다.

그러면 AI에게 단순히

> “RAT downgrade 로직 분석해”

라고 하는 게 아니라,

```text
RAT Downgrade
→ Entry가 어디인지
→ 어떤 context가 필요한지
→ 관련 module이 무엇인지
→ boundary output이 무엇인지
```

를 이미 map으로 제공할 수 있습니다.

그러면 앞에서 얘기한 Harness와 연결됩니다.

```text
Domain Map
= 무엇을 검증해야 하는지 알려줌

Harness
= 그 domain에 input을 넣고 실행함

Trace / Output
= 결과를 관찰함

Expected Result
= 검증 기준
```

즉 Lauren의 구조를 modem 식으로 바꾸면:

```text
Feature Map
      +
Control Skill
      +
Verification
```

이

```text
Protocol Domain Map
      +
Scenario Harness
      +
Boundary Verification
```

이 되는 겁니다.

그래서 **Feature Map은 테스트 코드가 아니라 Agent에게 주는 “제품/기능에 대한 구조화된 지식”**이라고 이해하면 가장 정확합니다. [Prepublish](https://prepublish.ai/youtube-transcript/NjoZoUm85x0?utm_source=chatgpt.com)




-------------------------

맞아요. 철자는 **governance**가 맞습니다.

그리고 륜님 해석이 핵심을 잘 짚었습니다. 이 영상이 단순히 “AI Agent를 잘 쓰는 법”으로 회자되는 게 아니라, 사실은 더 큰 엔지니어링 원칙을 보여주기 때문이라고 볼 수 있습니다.

좋은 결과물은 단순히 좋은 모델이나 좋은 개발자 한 명으로 나오는 게 아니라,

```text
Best Practice
+
Clear Governance
+
Verification
+
Automation
+
Feedback Loop
```

가 같이 있어야 합니다.

여기서 **Best Practice**는:

> “어떻게 일하는 것이 좋은가”

에 대한 축입니다.

예를 들면:

- 테스트를 먼저 설계한다
- 반복되는 문제는 자동화한다
- boundary를 명확히 한다
- 관찰 가능한 evidence를 남긴다
- regression을 방지한다

반면 **Governance**는:

> “그 좋은 방식이 실제로 계속 지켜지도록 어떤 규칙과 통제 구조를 둘 것인가”

에 더 가깝습니다.

예를 들면:

```text
이 변경은 어떤 test를 통과해야 하는가?
누가 merge할 수 있는가?
어떤 lint rule은 반드시 지켜야 하는가?
어떤 verification이 없으면 PR을 막을 것인가?
어떤 실패는 자동으로 reject할 것인가?
```

이런 것이 governance입니다.

그래서 Lauren 영상의 흐름도 사실:

```text
좋은 Prompt
    ↓
X
```

가 아니라

```text
좋은 Engineering Practice
    ↓
Verification
    ↓
Rule / Test / CI
    ↓
Governance
    ↓
신뢰 가능한 시스템
```

으로 보는 게 더 정확합니다.

특히 **auto merge**가 가능한 이유도 governance가 있기 때문입니다.

아무 코드나 자동 merge하는 게 아니라,

```text
Test PASS
Lint PASS
Verification PASS
정해진 조건 충족
```

같은 명확한 gate가 있어서,

> “이 조건을 만족하면 merge해도 된다”

라는 **governance rule**이 시스템에 박혀 있는 겁니다.

그래서 좋은 엔지니어링 조직은 사람에게 이렇게 말하지 않습니다.

```text
"잘 알아서 해주세요."
```

대신:

```text
Best Practice를 정의하고
    ↓
그걸 Test / CI / Lint / Review Rule로 구조화하고
    ↓
그 규칙을 시스템이 강제하게 한다
```

로 갑니다.

이게 륜님이 LTE Security 쪽에서 생각한 방식과도 다시 연결됩니다.

```text
공격마다 rule 계속 추가
    ↓
복잡도 폭증
```

보다

```text
Protocol principle 정의
    ↓
Invariant 정의
    ↓
Detection logic
    ↓
Policy / governance
```

로 가는 게 더 좋은 구조죠.

한 문장으로 압축하면:

> **Best Practice는 “어떻게 잘할 것인가”이고, Governance는 “그 좋은 방식이 계속 지켜지게 어떻게 통제할 것인가”입니다.**

그리고 Lauren 영상이 인상적인 이유도 바로 이 두 가지를 AI 개발에 실제 시스템으로 구현해서 보여주기 때문이라고 볼 수 있습니다.


----------------


네. 륜님이 지금 영상에서 느낀 **“좋은 엔지니어링 = 개인의 감이 아니라, 좋은 원칙·검증·자동화·governance를 시스템으로 만든 것”**이라는 관점에는 계보가 꽤 분명합니다.

제가 특히 추천할 사람은 아래 6명입니다.

1. **W. Edwards Deming** — 가장 뿌리에 가까운 인물입니다. 소프트웨어 엔지니어는 아니지만, 품질을 개인의 주의력 문제가 아니라 **시스템과 프로세스의 문제**로 봤습니다. 지속적 개선, 변동 관리, 피드백 루프 같은 사고를 널리 퍼뜨렸습니다. 지금 우리가 말하는 “사람이 잘하길 기대하지 말고 시스템을 개선하라”는 사고의 원류 중 하나입니다. [The W. Edwards Deming Institute](https://deming.org/deming-the-man/?utm_source=chatgpt.com)

2. **David Parnas** — 소프트웨어 구조 측면에서 매우 중요합니다. 핵심은 **information hiding과 modularity**입니다. 시스템을 잘 나누고, 모듈 사이 boundary를 명확히 해야 변경 영향이 제한된다는 철학이죠. 륜님이 지금 생각하는 **“특정 modem domain을 boundary로 감싸고 input/output으로 검증한다”**는 접근과 직접 연결됩니다.

3. **Kent Beck** — TDD와 Extreme Programming의 대표 인물입니다. 핵심 철학은 “코드를 쓴 뒤 사람이 감으로 확인”하는 것이 아니라 **작은 단위의 자동 테스트를 feedback loop로 만든다**는 것입니다. 즉 verification을 개발 workflow 안으로 집어넣은 사람이라고 볼 수 있습니다.

4. **Jez Humble** — Continuous Delivery 쪽에서 아주 중요합니다. 변경을 작게 만들고, build/test/deploy를 자동화하고, **매 변경마다 자동 검증 가능한 pipeline**을 만드는 접근을 체계화했습니다. Lauren 영상의 PR verification → CI → auto merge와 가장 직접적으로 이어지는 계보 중 하나입니다.

5. **Ben Treynor Sloss** — Google SRE를 만든 사람입니다. 이 사람은 륜님이 지금 느끼는 철학과 특히 잘 맞습니다. 그의 핵심 생각은 반복적인 사람이 하는 운영 작업을 **software engineering과 automation으로 대체**하는 것입니다. Google은 SRE를 “software engineer가 operations function을 설계하면 어떻게 되는가”라는 식으로 설명하고, 사람이 반복적으로 하는 toil을 자동화하는 것을 매우 중요하게 봅니다. [Google SRE](https://sre.google/sre-book/introduction/?utm_source=chatgpt.com)

6. **John Ousterhout** — 륜님에게 개인적으로 가장 추천하고 싶습니다. *A Philosophy of Software Design* 저자입니다. 이 사람은 좋은 설계의 핵심을 **complexity 관리**로 봅니다. 특히 complexity의 징후를 `change amplification`, `cognitive load`, `unknown unknowns`로 설명하고, 의존성을 줄이고 구조를 명확하게 만들어야 한다고 강조합니다. [Stanford University](https://web.stanford.edu/~ouster/cgi-bin/book.php?trk=public_post_comment-text\&utm_source=chatgpt.com)

이걸 하나의 계보로 보면 꽤 재미있습니다.

```text
Deming
"품질은 개인의 노력보다 시스템의 결과다"
        ↓
Parnas
"Boundary와 module을 잘 나눠라"
        ↓
Kent Beck
"검증을 빠른 automated feedback loop로 만들어라"
        ↓
Jez Humble
"그 검증을 delivery pipeline 전체로 확장하라"
        ↓
Google SRE / Ben Treynor Sloss
"반복되는 인간 작업을 engineering과 automation으로 제거하라"
        ↓
Lauren Tan
"AI Agent도 사람이 매번 검증하지 않도록
verification system을 만들어라"
```

그래서 Lauren의 발표가 완전히 새로운 철학이라기보다, **오랫동안 발전해온 좋은 소프트웨어 엔지니어링 원칙을 AI Agent 시대에 다시 적용한 것**이라고 보는 게 더 정확합니다.

특히 륜님이 지금 고민하는 Modem Verification Framework에도 그대로 이어집니다.

```text
Parnas
→ Domain boundary

Kent Beck
→ Automated test / feedback

Jez Humble
→ Regression pipeline

Google SRE
→ Human verification toil 제거

Lauren
→ Agent가 스스로 verification

륜님 아이디어
→ RF 장비실 반복 검증을
   boundary-driven protocol verification으로 전환
```

그리고 하나만 먼저 읽는다면 저는 **John Ousterhout의 _A Philosophy of Software Design_**을 추천합니다. “왜 모든 걸 다 재현하지 말고 domain을 잘 잘라야 하는가”, “왜 dependency와 interface가 중요한가”가 지금 륜님 고민과 거의 바로 연결됩니다. 그다음 Google의 **SRE Book**을 보면 “왜 사람의 반복 확인을 자동화 시스템으로 바꿔야 하는가”가 연결됩니다. [Google SRE](https://sre.google/sre-book/introduction/?utm_source=chatgpt.com)
