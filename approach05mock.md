맞습니다. 그게 핵심입니다. **모뎀 전체를 replay하려고 하면 사실상 mini-modem을 다시 만드는 수준**이 돼서 범위가 폭발합니다. 그래서 반드시 **domain boundary를 먼저 자르고, 그 domain 안에서 entry와 end를 명확히 잡는 방식**으로 가야 합니다.

예를 들어 Cell Reselection만 본다면:

```text
[Entry]
Measurement indication 수신
    ↓
Reselection 관련 logic만 추적
    ↓
Candidate 평가
    ↓
Threshold / Priority / Timer 판단
    ↓
Target 결정
    ↓
[End]
"이 cell로 reselection 요청" primitive 발생
```

여기서 중요한 건 **그 뒤 실제 RF tune, sync, camp 완료까지 replay할 필요는 없다는 것**입니다.

즉 boundary를 이렇게 자르는 겁니다.

```text
실제 시스템 전체

L1 measurement
    ↓
RRC mobility logic     ← 내가 검증할 domain
    ↓
L1 cell selection command
    ↓
RF / DSP / HW 동작
```

테스트에서는:

```text
Mock Measurement
    ↓
[ RRC mobility logic ]
    ↓
Expected Cell Selection Command
```

만 검증합니다.

이렇게 보면 **Entry와 End는 결국 Test Boundary**입니다.

좋은 boundary는 보통:

```text
Entry
= 내가 제어 가능한 input이 들어오는 지점

End
= 내가 확인하고 싶은 decision/output이 나오는 지점
```

입니다.

예를 들어 NAS Security Detection이라면:

```text
Entry:
Reject / RAT change / Cell info / Security context event

        ↓

내가 담당하는 detection logic

        ↓

End:
Network Security Event 생성 여부
Category / Reason / Cell Info
```

여기까지만 보면 됩니다.

그 아래 AP HAL 전달이나 framework 동작까지 전부 replay할 필요는 없습니다.

이렇게 domain 중심으로 보면 전체 구조는:

```text
Outside Domain
      ↓
Mock / Stub
      ↓
========================
     Target Domain
========================
      ↓
Observed Output
      ↓
Mock / Stub
```

가 됩니다.

그리고 이 방식의 장점이 아주 큽니다. **Mock해야 할 것도 급격히 줄어듭니다.**

모뎀 전체를 replay하려고 하면:

```text
L1
DSP
RF
Scheduler
Timer
IPC
RRC
NAS
SIM
Network
...
```

를 다 만들어야 하지만,

domain을 잘 자르면:

```text
Required Input A
Required Input B
Required State C
Required Timer D
```

정도만 재현하면 될 수 있습니다.

그래서 실제 framework 개발에서 제일 먼저 해야 할 일이 코드 작성보다 오히려 이겁니다.

```text
Scenario 정의
    ↓
Target Domain 정의
    ↓
Entry Point
    ↓
Exit / End Point
    ↓
Required Inputs
    ↓
Internal State
    ↓
Expected Outputs
```

저라면 각 테스트를 아예 이런 템플릿으로 관리할 것 같습니다.

```text
Scenario:
LTE Cell Reselection

Target Domain:
RRC Mobility / Reselection Decision

Entry:
Measurement Indication

Exit:
Cell Reselection Request

Inputs:
- Serving RSRP
- Neighbor RSRP
- Priority
- SIB parameter
- Timer condition

Initial State:
- Current cell
- RRC idle
- Candidate list

Expected:
- Candidate B selected
- T_reselection started
- Target Cell B requested

Out of Scope:
- RF tuning
- Synchronization
- Actual camping
- DSP behavior
```

여기서 **Out of Scope를 명시하는 게 굉장히 중요합니다.**

그래야 framework가 계속 커지는 걸 막을 수 있습니다.

즉 원칙은:

> **전체 시스템을 simulation하려 하지 말고, 내가 검증하려는 decision domain의 input과 output만 재현한다.**

그리고 이게 사실 Unit/Component Test 설계의 본질이기도 합니다.

조금 더 정확히 말하면 륜님이 만들려는 것은 **Full Modem Simulator**가 아니라,

> **Domain-scoped Protocol Verification Harness**

에 가깝습니다.

이렇게 정의하면 방향이 아주 명확해집니다.


------------



**Harness**는 테스트 문맥에서 쉽게 말하면:

> **테스트 대상 코드를 실행시키고, 필요한 입력을 넣고, 결과를 확인하게 해주는 테스트용 껍데기/실행 환경**

입니다.

륜님 상황에 맞게 보면:

```text
[Test Harness]
   ↓
Mock Input 주입
   ↓
Target Modem Logic 실행
   ↓
State / Output 수집
   ↓
Expected Result와 비교
   ↓
PASS / FAIL
```

예를 들어 Cell Reselection이면:

```text
Harness가 하는 일

1. 초기 state 설정
   - Serving Cell
   - RRC state
   - priority

2. Mock measurement 입력
   - serving RSRP
   - neighbor RSRP

3. Entry function 호출

4. Timer/event 진행

5. 결과 확인
   - target cell
   - state transition
   - reselection request

6. PASS / FAIL 판정
```

즉 **Harness 자체가 테스트 대상은 아닙니다.** 테스트 대상 코드를 주변에서 감싸서 실행 가능하게 만들어주는 도구입니다.

륜님이 생각하는 구조로 표현하면:

```text
Mock Data
   ↓
┌──────────────────────────┐
│      Test Harness        │
│                          │
│  Entry 호출              │
│  Timer/Event 제어        │
│  State 관찰              │
│  Output 수집             │
└────────────┬─────────────┘
             ↓
      Modem Target Logic
```

그래서 **`Domain-scoped Protocol Verification Harness`**라고 하면,

> **특정 protocol domain만 잘라서 입력을 주입하고 로직을 실행하고 결과를 검증하는 테스트 환경**

이라는 뜻입니다.

가장 짧게 기억하면:

> **Harness = 테스트 대상 코드를 실제로 돌려보기 위한 테스트 실행 틀**

-------------



맞습니다. 정확히 그 개념입니다.

> **특정 Domain을 하나의 검증 대상 블록으로 보고, 그 Domain의 Boundary Interface를 통해 Input을 넣고 Output을 관찰해서 내부 Logic을 검증하는 것**

입니다.

구조로 보면:

```text
             Outside World
                  │
                  │ Input
                  ▼
        ┌───────────────────┐
        │   Target Domain   │
        │                   │
        │   Internal Logic  │
        │   State Machine   │
        │   Decision Logic  │
        └───────────────────┘
                  │
                  │ Output
                  ▼
             Outside World
```

테스트에서는 Outside World를 실제 RF/HW가 아니라 **Harness가 대신**합니다.

```text
        Test Harness
             │
             │ Mock Input
             ▼
        ┌───────────────────┐
        │   Target Domain   │
        │                   │
        │   Real SW Logic   │
        └───────────────────┘
             │
             │ Observed Output
             ▼
        Test Harness
             │
             ▼
       Expected와 비교
             │
        PASS / FAIL
```

예를 들어 Cell Reselection Domain이라면:

```text
Boundary Input
- Measurement indication
- Serving / Neighbor RSRP
- SIB parameter
- Priority
- Timer event
- Current state

        ↓

[ Reselection Domain ]

        ↓

Boundary Output
- Candidate selection
- State transition
- Timer start/stop
- Cell reselection request
```

여기서 중요한 점은 **Domain 내부 구현은 실제 modem code를 그대로 사용한다는 것**입니다.

Mock하는 건 주로 바깥쪽입니다.

```text
실제 환경

L1 / RF
   ↓
RRC Reselection Logic
   ↓
L1 / Cell Selection
```

을 테스트에서는:

```text
Mock L1 Input
   ↓
RRC Reselection Logic   ← 실제 production code
   ↓
Mock / Captured Output
```

으로 만드는 겁니다.

그래서 이 접근의 핵심은 결국 **Boundary를 얼마나 잘 정의하느냐**입니다.

좋은 Domain Test는:

```text
명확한 Entry
+
명확한 Exit
+
최소한의 Input
+
필요한 Initial State
+
관찰 가능한 Output
```

이 있어야 합니다.

그리고 륜님이 지금 이야기한 방향에서 한 단계 더 중요한 포인트가 있습니다.

> **Call flow를 완벽하게 재현하는 것이 목표가 아니라, Boundary 안의 production logic이 동일한 Input에 대해 동일한 Decision/Output을 만드는지 검증하는 것이 목표입니다.**

이렇게 정의하면 범위가 훨씬 명확해집니다.

즉, 우리가 만들려는 것은 사실상:

> **Boundary-driven Domain Verification**

이라고 표현해도 상당히 정확합니다.


