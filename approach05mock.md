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
