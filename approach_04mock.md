네. 방향이 맞습니다. 특히 **객체지향 구조보다 C-style function 중심의 modem stack**이라면, 오히려 처음에는 **call flow / call tree + state/context + external input** 관점으로 해부하는 게 더 자연스럽습니다.

다만 하나 보완하면, **call tree만 완성하면 부족하고 `data flow`와 `state dependency`까지 같이 봐야 합니다.** 모뎀 프로토콜 로직은 보통 함수 호출 자체보다 `context`, `global state`, `timer`, `message`, `callback/event`가 결과를 크게 좌우하기 때문입니다.

가장 현실적인 분석 단위는 이겁니다.

```text
Feature / Scenario
    ↓
Entry Event
    ↓
Call Flow
    ↓
State / Context Read & Write
    ↓
External Input
    ↓
Decision Point
    ↓
Output / Next Event
```

예를 들어 Cell Reselection이라면 단순히:

```text
funcA()
  → funcB()
      → funcC()
```

만 보면 부족합니다.

실제로는 이런 정보가 필요합니다.

```text
Measurement Indication
    ↓
reselection_eval()
    ↓
reads:
  serving_rsrp
  neighbor_rsrp
  priority
  sib_params
  mobility_state

    ↓
ranking_calc()

    ↓
decision:
  candidate_valid?
  threshold_met?
  timer_expired?

    ↓
writes:
  target_cell
  reselection_state

    ↓
send_cell_change_req()
```

즉 **call tree + read/write data map**을 같이 만들어야 합니다.

그리고 륜님 말씀대로 결국 replay/unit test를 하려면 필요한 input은 mock으로 만들어야 합니다. 다만 처음부터 “모든 input을 mock”하려고 하면 너무 큽니다.

좋은 순서는:

```text
1. Scenario 하나 선택
2. Entry function 찾기
3. 관련 call flow 추적
4. 각 function이 참조하는 input/state 목록화
5. 실제 log와 source를 비교해서
   "이 시나리오에서 실제 필요한 값"만 추출
6. 그 값부터 mock
```

입니다.

예를 들어 이런 식으로 정리하면 좋습니다.

```text
Scenario: LTE Cell Reselection

Entry:
  meas_ind_handler()

Required Inputs:
  serving.rsrp
  neighbor[].rsrp
  serving.priority
  neighbor[].priority
  qRxLevMin
  threshXHigh
  tReselection
  current_state

Internal State:
  reselection_timer
  candidate_list
  current_cell

Expected Output:
  target_cell
  state_transition
  lower_layer_request
```

그러면 test harness는 이 정도로 시작할 수 있습니다.

```text
setup_context()
    ↓
inject_measurement()
    ↓
call_entry_function()
    ↓
advance_timer()
    ↓
check_state()
    ↓
check_output()
```

C-style code라면 오히려 이런 harness를 만들기 쉬운 부분도 있습니다. 함수 경계와 구조체 기반 context가 명확하면:

```text
Function
+ Input Struct
+ Global/Context Struct
+ Output/Event
```

로 분해할 수 있기 때문입니다.

하지만 주의해야 할 게 있습니다. 모뎀 코드는 흔히 call flow가 순수한 tree가 아닙니다.

예를 들어:

```text
Task A
  ↓
Message Queue
  ↓
Task B
  ↓
Timer Callback
  ↓
Task C
  ↓
Primitive
  ↓
다시 Task A
```

처럼 **event-driven graph**가 됩니다.

그래서 정확한 표현은 `call tree`보다:

> **Call Flow + Event Flow + State Flow**

가 더 맞습니다.

최종적으로는 이런 그림을 만드는 게 이상적입니다.

```text
             [Measurement Event]
                     ↓
              handler_func()
                     ↓
               eval_func()
               /       \
      read context      read SIB
            ↓              ↓
        decision_func()
              ↓
       start timer/event
              ↓
        timer callback
              ↓
       target_select()
              ↓
        output primitive
```

그리고 각 노드 옆에:

```text
Input:
State Read:
State Write:
Output:
```

를 붙이는 겁니다.

이렇게 되면 AI도 훨씬 잘 도와줄 수 있습니다. 예를 들어 full source를 기준으로:

```text
"이 entry function에서 시작해서
cell reselection decision까지
모든 direct/indirect call을 추적해줘."

"각 함수가 읽고 쓰는 context field를 정리해줘."

"외부 dependency와 pure logic을 분리해줘."

"이 scenario를 replay하려면
최소한 어떤 input을 mock해야 하는지 알려줘."
```

같은 요청이 가능합니다.

핵심은 이겁니다.

> **목표는 모든 함수의 call tree를 완벽히 그리는 게 아니라, 특정 scenario를 재현하는 데 필요한 최소 실행 경로와 최소 input set을 찾는 것입니다.**

그게 훨씬 현실적이고, framework 구축에도 직접 연결됩니다.
