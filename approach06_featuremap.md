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
