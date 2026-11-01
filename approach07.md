맞아요. 지금 이해가 정확합니다.

Lauren 쪽에서는 처음에 **“화면 제어”라는 특정 domain**을 잡고, 그 domain 안에서 재사용 가능한 skill을 만든 겁니다. 그래서 어떤 시나리오가 들어오면 Agent가 매번 임의로 화면을 다루는 게 아니라, 이미 정의된 **화면 제어 skill**을 사용해서 user-facing interface를 조작한 거죠.

구조로 보면:

```text
Scenario
  ↓
Feature / Domain 판단
  ↓
해당 Domain Skill 선택
  ↓
화면 제어 Skill 실행
  ↓
User Input Interface 조작
  ↓
결과 관찰
  ↓
Verification
```

예를 들어 시나리오가

```text
"PR tab이 느리다"
```

라면 Agent는:

```text
Feature Map
→ PR tab 위치 확인

Control Skill
→ 창 열기
→ sidebar 클릭
→ PR tab 이동

Observation
→ trace / UI state / latency 확인

Verification
→ 실제 문제 재현 여부 판단
```

이런 식으로 움직입니다.

그래서 여기서 **Skill은 domain별 capability 묶음**이라고 보면 이해가 쉽습니다.

예를 들면 Cursor 같은 제품이라면:

```text
UI Control Skill
- click
- type
- open panel
- navigate
- screenshot

Performance Skill
- start trace
- stop trace
- inspect flame graph
- collect timing

Git Skill
- branch
- diff
- commit
- PR

Test Skill
- build
- lint
- test
```

처럼 나눌 수 있습니다.

즉 모든 것을 하나의 거대한 skill로 만들기보다,

> **특정 domain에서 반복되는 control / observation / verification 방법을 skill로 만든다**

는 접근입니다.

그리고 륜님 modem 아이디어에도 그대로 대응시킬 수 있습니다.

예를 들어:

```text
RRC Mobility Skill
- measurement input inject
- timer advance
- cell context setup
- state observe
- reselection output capture

NAS Security Skill
- NAS PDU inject
- security context setup
- reject event inject
- RAT state setup
- security event observe
```

처럼 갈 수 있습니다.

즉 modem에서도 시나리오가 들어오면:

```text
Scenario
→ Target Domain 결정
→ Domain Skill 선택
→ Boundary Input 주입
→ Production Logic 실행
→ Boundary Output 관찰
→ Verification
```

이 구조가 됩니다.

그래서 륜님 말대로 **화면 제어는 결국 user input interface를 자동화한 domain skill**이고, modem에서는 그에 해당하는 것이 **protocol boundary input/output을 제어·관찰하는 domain skill**이 될 수 있습니다.

한 문장으로 정리하면:

> **Skill은 “무엇을 테스트할 것인가”가 아니라, 특정 domain을 어떻게 제어하고 관찰하고 검증할 것인가를 재사용 가능하게 만든 실행 capability입니다.**

----------------

이 장면은 지금까지 얘기한 내용의 핵심을 그림으로 보여주는 부분이에요.

Lauren이 말하는 건 대략 이런 뜻입니다.

> **“초기에는 Agent를 거의 믿지 못해서 내가 모든 과정에 직접 개입했고, Agent가 하는 사소한 행동까지 하나하나 확인했다.”**

화면의 영어 문장도 그 의미입니다.

> “In the early days … I was super in the loop and I was basically nitpicking my agents to an extreme degree.”

여기서 **in the loop**는 “모든 과정에 사람이 직접 끼어 있다”는 뜻이고, **nitpicking**은 “아주 사소한 것까지 꼬치꼬치 확인한다”는 뜻입니다.

그래서 그림의 세로축 `trust`는 **Agent에 대한 신뢰 수준**이고, 곡선은 시간이 지나면서 신뢰가 높아지는 모습을 표현한 겁니다.

초기에는 이런 상태였던 거죠.

```text
Agent 작업
   ↓
Human 확인
   ↓
Agent 작업
   ↓
Human 확인
   ↓
Agent 수정
   ↓
Human 확인
```

즉 **Human-in-the-loop가 아주 강한 상태**입니다.

그런데 Lauren이 verification skill, feature map, CLI, Playwright, trace 같은 걸 만들어가면서 점점:

```text
Agent
 ↓
작업
 ↓
직접 실행
 ↓
직접 관찰
 ↓
직접 검증
 ↓
수정
 ↓
재검증
```

이 가능해졌고, 그만큼 사람의 개입을 줄일 수 있게 됩니다.

그래서 이 그래프의 의미는 단순히

> “AI가 점점 똑똑해졌다”

가 아니라,

> **“Agent가 스스로 검증할 수 있는 시스템을 만들어가면서 신뢰가 축적됐다.”**

에 가깝습니다.

그리고 륜님이 지금 modem verification 쪽에서 생각하는 것도 완전히 같은 구조예요.

초기에는:

```text
Patch
 ↓
Engineer가 장비실 가서 직접 확인
 ↓
Log 확인
 ↓
판단
```

인데, boundary-driven verification harness가 생기면:

```text
Patch
 ↓
Scenario Replay
 ↓
Boundary Input 주입
 ↓
Production Logic 실행
 ↓
Boundary Output 확인
 ↓
PASS / FAIL
```

이 되고, 점점 사람의 개입을 줄일 수 있습니다.

즉 이 장면의 핵심 메시지는:

> **Trust는 처음부터 주어지는 게 아니라, 반복 가능한 verification system을 통해 쌓이는 것이다.**

그리고 그 결과가 결국 **더 적은 human-in-the-loop → 더 높은 병렬성 → 더 높은 automation**으로 이어지는 겁니다.




-----------------


맞아요. 여기서 Lauren이 말하는 **eval**은 아주 쉽게 말하면:

> **“내가 만든 Agent Skill이 실제로 Agent의 행동을 원하는 방향으로 바꾸는지 테스트하는 것”**

입니다.

그래서 **“Agent를 위한 unit test”**라고 표현한 거예요. [Use Transcribe](https://www.usetranscribe.io/yt/fiIW_zS5CV4/spacex-ai-engineer-tan?utm_source=chatgpt.com)

예를 들어 일반 코드의 unit test는:

```text
함수에 input을 넣는다
→ output이 예상과 같은지 본다
```

입니다.

Agent eval은:

```text
Agent에게 task를 준다
→ 특정 skill을 사용하게 한다
→ Agent가 실제로 어떻게 행동했는지 본다
→ 내가 정의한 기준을 만족했는지 평가한다
```

가 됩니다.

예를 들어 앞에서 Lauren이 만든 **“추측하지 말고 코드를 읽어라”** skill이 있다고 해보죠.

그 skill이 정말 효과가 있는지 확인하려면 eval을 이렇게 만들 수 있습니다.

```text
Task:
"이 feature가 왜 느린지 원인을 찾아라"

Expected behavior:
- 관련 source를 search한다
- 실제 code를 read한다
- call path를 확인한다
- 근거 없이 추측하지 않는다
- verification을 수행한다
```

그리고 Agent 여러 개에게 같은 task를 시켜봅니다.

```text
Agent A
→ source 읽음
→ trace 확인
→ 근거 제시
→ PASS

Agent B
→ 코드 안 읽고 추측
→ FAIL

Agent C
→ 일부만 확인
→ PARTIAL
```

이게 eval입니다.

그래서 일반 unit test와 대응시키면 아주 이해가 쉽습니다.

```text
Software Unit Test
-------------------------
Input
→ Function
→ Output
→ Assert

Agent Eval
-------------------------
Task / Prompt
→ Agent + Skill
→ Behavior / Artifact
→ Rubric로 평가
```

여기서 **rubric**이 중요합니다.

Rubric은:

> **“무엇을 잘했다고 볼 것인가?”에 대한 채점 기준**

입니다.

Lauren은 main coordinator agent가 먼저 rubric을 만들고, 그다음 여러 sub-agent를 실행해서 skill이 실제로 그 기준을 만족하는지 평가한다고 설명합니다. [Use Transcribe](https://www.usetranscribe.io/yt/fiIW_zS5CV4/spacex-ai-engineer-tan?utm_source=chatgpt.com)

예를 들어:

```text
Rubric

1. Relevant source를 실제로 읽었는가?      2점
2. 추측 대신 evidence를 제시했는가?         2점
3. 실제 app을 실행했는가?                    2점
4. 문제를 reproduce했는가?                   2점
5. 수정 후 다시 verification했는가?          2점

Total = 10
```

이런 식입니다.

그리고 Lauren이 꽤 흥미로운 얘기를 하나 합니다.

Agent가 **“내가 지금 평가받고 있다”**는 걸 알면 행동이 달라질 수 있어서, sub-agent에게 eval이라는 걸 티 내지 않도록 별도 directory를 평범한 이름으로 만든다고 합니다. 즉 실제 업무 상황과 비슷하게 테스트하려는 거죠. [Use Transcribe](https://www.usetranscribe.io/yt/fiIW_zS5CV4/spacex-ai-engineer-tan?utm_source=chatgpt.com)

또 Cursor는 여러 모델을 지원하니까:

```text
같은 Skill

→ Model A에서 Eval
→ Model B에서 Eval
→ Model C에서 Eval
```

해서 skill이 특정 모델에서만 잘 되는 게 아니라 여러 모델에서도 안정적으로 동작하는지 볼 수 있습니다. [Use Transcribe](https://www.usetranscribe.io/yt/fiIW_zS5CV4/spacex-ai-engineer-tan?utm_source=chatgpt.com)

그리고 **20:16 이후** Lauren이 강조하는 게 하나 더 있습니다.

Skill을 만드는 초기에는 Agent를 수동적으로 보지 말고, pair programming에서 옆자리 동료 코드를 보는 것처럼:

```text
왜 저 tool을 안 썼지?
왜 source를 안 읽었지?
왜 여기서 추측했지?
왜 verification 없이 끝냈지?
```

를 계속 관찰하라는 겁니다. 그렇게 반복 failure pattern을 찾고, 그걸 skill로 만들고, 다시 eval로 테스트합니다. [Use Transcribe](https://www.usetranscribe.io/yt/PaPpyQocMww/spacex-ai-engineer-tan?utm_source=chatgpt.com)

전체 루프는 이렇게 됩니다.

```text
Agent 행동 관찰
    ↓
Failure pattern 발견
    ↓
Skill 작성 / 수정
    ↓
Eval 실행
    ↓
Score 확인
    ↓
Skill 개선
    ↓
다시 Eval
```

Lauren은 이걸 **hill climbing**한다고 표현합니다. 즉 eval score를 보면서 skill을 조금씩 개선해서 점수를 끌어올리는 겁니다. 다른 모델의 judge agent를 써서 첫 번째 모델의 편향도 교차검증한다고 설명합니다. [Use Transcribe](https://www.usetranscribe.io/yt/PaPpyQocMww/spacex-ai-engineer-tan?utm_source=chatgpt.com)

이게 륜님 modem verification하고도 아주 재미있게 연결됩니다.

```text
Modem SW

Scenario Input
→ Target Domain
→ Boundary Output
→ Expected Result
→ PASS / FAIL
```

이게 **software behavior verification**이라면,

```text
Agent

Engineering Task
→ Agent + Modem Skill
→ 실제 행동
→ Rubric
→ PASS / FAIL
```

은 **Agent behavior verification**입니다.

즉 둘 다 본질은 같습니다.

> **“잘할 것이라고 믿지 말고, 반복 가능한 test로 증명하라.”**

그래서 Lauren의 eval 개념을 한 문장으로 기억하면:

> **Eval = Agent가 우리가 기대한 방식으로 행동하는지 검증하는 자동화된 behavioral test**

입니다.

