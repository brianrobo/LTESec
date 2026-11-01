# Modem Protocol Scenario Verification Approach

이 아이디어는 상당히 타당하다.

다만 정확히는

> **“Unit Test만으로 실제 RF 시나리오를 대체한다”**

기보다,

> **장비실에서 확보한 실제 Protocol Scenario를 Software-level Replay / Component Test로 재현해서, 대부분의 Regression Verification을 개발 환경으로 끌어오는 접근**

으로 정의하는 것이 더 좋다.

핵심 구조는 다음과 같이 볼 수 있다.

```text
실제 RF 장비 / Shield Room
        ↓
특정 시나리오 수행
(HO / Cell Reselection / RF 변화 / Reject 등)
        ↓
Golden Log / Trace 확보
        ↓
관련 모듈 / 함수 / State Transition 분석
        ↓
외부 Input과 내부 Dependency 분리
        ↓
Replay 가능한 Test Vector 생성
        ↓
Modem SW-level Scenario Test
        ↓
Patch 이후 자동 Regression Verification
```

이것은 앞에서 이야기한 Lauren의 **Verification 환경을 만드는 접근**과도 아주 비슷하다.

Lauren은 사람이 매번 UI를 직접 만져보지 않고,

```text
실제 앱
→ observable data
→ CLI / Playwright
→ 자동 verification
```

으로 옮겼다.

Modem 쪽에서는,

```text
실제 RF 환경
→ 실제 modem trace
→ scenario input model
→ software replay harness
→ 자동 verification
```

으로 옮기는 것이다.

---

# 특히 좋은 점

현재 가장 큰 병목은 **RF Environment 자체가 Scarce Resource**라는 점이다.

```text
장비실 이동
→ 장비 설정
→ Shield Room
→ RF Scenario 실행
→ Log 확보
→ 인터넷 제한
→ 분석
```

이 과정을 모든 Patch마다 반복하기는 어렵다.

그런데 실제 장비에서 한 번 검증된 Scenario를 Software Test로 모델링하면,

```text
Patch
 ↓
Build
 ↓
Scenario Replay
 ↓
Expected State / Output 비교
 ↓
Pass / Fail
```

가 가능해진다.

이렇게 되면 장비실은

> **매번 모든 변경을 검사하는 장소**

가 아니라,

> **Software Test Model을 Calibration하고 새로운 Scenario의 Ground Truth를 만드는 장소**

로 역할이 바뀔 수 있다.

이것이 상당히 중요한 변화다.

---

# 다만 "Unit Test"라는 표현은 조금 넓게 보는 것이 좋다

예를 들어 Handover를 생각하면 단일 함수 하나만 검증해서는 부족할 가능성이 높다.

```text
Measurement Input
   ↓
RRC/NAS Event
   ↓
State Machine
   ↓
Timer
   ↓
Cell Information
   ↓
Mobility Decision
   ↓
Lower-layer Command
```

처럼 여러 모듈이 연결되기 때문이다.

따라서 실제로는 3단계 정도로 나누는 것이 좋다.

```text
Level 1
Pure Unit Test
→ 개별 함수 / 알고리즘

Level 2
Component / Module Test
→ NAS / RRC / Mobility State Machine

Level 3
Scenario Replay Test
→ 실제 장비 Log 기반 End-to-End SW Flow
```

지금 이야기한 핵심 아이디어는 **Level 2~3**, 특히 `Scenario Replay Test`에 더 가깝다고 볼 수 있다.

---

# 예를 들어 Cell Reselection이라면

장비실에서 실제 Scenario를 하나 수행한다.

```text
Serving Cell
RSRP = -90 dBm

Neighbor Cell
RSRP = -105 dBm

        ↓

Neighbor RF 증가

        ↓

Neighbor
RSRP = -82 dBm

        ↓

Reselection Criteria 만족

        ↓

Cell Reselection
```

여기서 Modem Trace를 확보한다.

그 다음 단순히 최종 결과만 보는 것이 아니라 **필요한 Input을 역으로 추출**한다.

```text
Input
 ├─ Serving RSRP / RSRQ
 ├─ Neighbor RSRP / RSRQ
 ├─ Srxlev
 ├─ Cell Priority
 ├─ Threshold
 ├─ Hysteresis
 ├─ Timer
 ├─ SIB Parameters
 └─ Current Mobility State
```

그리고 내부에서 지나가는 주요 Flow를 찾는다.

```text
Measurement Indication
        ↓
Reselection Evaluation
        ↓
Candidate Ranking
        ↓
Criteria Check
        ↓
Timer
        ↓
Target Selection
        ↓
Cell Change Request
```

그 다음 Test Harness에서 RF Chipset이 실제로 값을 주지 않아도,

```text
Mock Measurement Provider
        ↓
{
  serving_rsrp: -90,
  neighbor_rsrp: -82,
  priority: 7,
  ...
}
```

처럼 Input을 넣는다.

그러면 Expected Result를 정의할 수 있다.

```text
Expected

Candidate Cell = Cell B
Reselection Condition = TRUE
Target RAT = LTE
Selected PCI = xxx
State Transition = A → B → C
```

여기까지 만들어 놓으면 이후 Patch가 들어왔을 때

```text
Before Patch → PASS

Patch 적용

After Patch → Replay

Expected State Transition과 비교
```

가 된다.

이것이 바로 **Regression Verification**이다.

---

# 더 중요한 것은 Expected Result를 단순 최종 결과로만 만들지 않는 것이다

예를 들어

```text
PASS = Cell Reselection 성공
```

만 확인하면 문제가 숨어 있을 수 있다.

가능하면 여러 층을 봐야 한다.

```text
Input
 ↓
Decision
 ↓
State Transition
 ↓
Output
```

예:

```text
① Measurement Input 정상 수신

② Candidate Ranking
Cell B > Cell A

③ Reselection Criteria
TRUE

④ Timer
T_reselection Start

⑤ State
IDLE_SERVING
   →
RESELECTING

⑥ Target Selection
Cell B

⑦ 최종 Primitive / Command
정상 발생
```

그러면 Patch가 Behavior를 미묘하게 깨뜨려도 어디서 깨졌는지 바로 알 수 있다.

---

# 실제 장비 Log의 역할도 중요하다

여기서 실제 Log를 단순 참고자료로만 쓰지 않고,

> **Golden Trace**

또는

> **Reference Scenario**

로 만드는 것을 권장한다.

예를 들면:

```text
Scenario ID:
LTE_RESEL_001

Purpose:
Higher Priority LTE Neighbor Reselection

Real Equipment:
Anritsu / R&S / Keysight ...

Initial Condition:
Serving Cell A
Neighbor Cell B

Network Parameters:
...

RF Transition:
...

Expected Protocol Flow:
...

Relevant Modem Modules:
...

Required Input Vectors:
...

Expected State Transition:
...

Expected Output:
...

Reference Trace:
xxxx.log
```

이렇게 만들면 단순한 Log 하나가 아니라 **재현 가능한 Specification**이 된다.

---

# 이 접근에서 가장 조심해야 할 부분

Software Replay가 실제 장비 테스트를 완전히 대체한다고 보면 위험하다.

왜냐하면 실제 RF 환경에는 Software Mock에서 놓치기 쉬운 요소가 있기 때문이다.

```text
Real RF Timing
Layer 1 Behavior
Interrupt Timing
Scheduler
Race Condition
RF Calibration
DSP Interaction
Inter-layer Asynchronous Events
Timing Jitter
Hardware-specific Behavior
```

따라서 역할을 다음과 같이 나누는 것이 가장 좋다.

```text
                 ┌─────────────────────────┐
                 │ Real RF / Shield Room   │
                 │ Ground Truth            │
                 │ Final Validation        │
                 └────────────┬────────────┘
                              │
                      Golden Scenario
                              │
                              ▼
                 ┌─────────────────────────┐
                 │ SW Scenario Replay      │
                 │ Regression Verification │
                 │ Fast / Automated        │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │ Unit / Component Test   │
                 │ Logic Verification      │
                 └─────────────────────────┘
```

즉,

> **실장비 Test는 Ground Truth와 최종 검증**

> **Scenario Replay는 일상적인 Regression**

> **Unit Test는 세부 Logic 검증**

으로 역할을 나누는 것이다.

이 구조라면 현실적이다.

---

# LTE Security 연구에도 직접 연결된다

이 구조를 만들어 놓으면 Security Detection 개발에도 상당한 이점이 있다.

예를 들어 Downgrade Detection을 만들었다고 가정한다.

현재 방식이면:

```text
Security Logic Patch
    ↓
장비실
    ↓
Network Scenario 설정
    ↓
RAT Downgrade 유도
    ↓
Log 확인
```

을 계속 해야 한다.

하지만 Scenario Replay Framework가 있다면,

```text
Real Scenario Trace
     ↓
Relevant Inputs 추출

Reject
RAT Information
Cell Information
Security Context
Timer
Measurement
...

     ↓

Software Replay
     ↓
Security Detector

     ↓

Expected:
DOWNGRADE Event 발생
```

을 만들 수 있다.

그러면 이후 Detection Logic을 조금 수정할 때마다

```text
Downgrade Scenario 01
Downgrade Scenario 02
Normal Mobility Scenario 01
Normal Mobility Scenario 02
Weak RF Scenario
Reject Scenario
...
```

를 한꺼번에 돌릴 수 있다.

특히 중요한 것은 **Attack Test뿐 아니라 정상 Scenario도 같이 넣는 것**이다.

```text
Attack Scenario
→ Detection 해야 함

Normal Scenario
→ Detection 하면 안 됨
```

그래야 False Positive Regression도 잡을 수 있다.

이것은 Security에서는 특히 중요하다.

---

# 앞에서 말한 Invariant 접근과도 연결된다

단순히

```text
Test 001 = Attack A
Test 002 = Attack B
Test 003 = Attack C
```

만 만드는 것이 아니라,

상위 수준에서는

```text
Invariant:

"Legitimate Mobility로 설명되지 않는
Security Degradation은 없어야 한다."
```

라고 정의하고,

그 아래 여러 Scenario를 배치한다.

```text
Invariant:
No unexplained security degradation

 ├─ Normal LTE → LTE Mobility
 │      → No Alert
 │
 ├─ Weak RF LTE → 3G
 │      → No Alert
 │
 ├─ Network Commanded Legitimate Fallback
 │      → No Alert
 │
 ├─ Repeated Reject → 3G Forced Downgrade
 │      → Alert
 │
 └─ Abnormal LTE → 2G Transition
        → Alert
```

그러면 이 테스트 시스템이 단순 Regression Test를 넘어,

> **Protocol Invariant를 Executable Specification으로 만드는 시스템**

이 된다.

이것이 상당히 좋은 방향이다.

---

# 이 Approach를 한 줄로 정의한다면

> **실장비에서 관찰한 Protocol Behavior를 Golden Scenario로 만들고, 필요한 Modem Input과 State를 추출하여 Software에서 Replay 가능한 Executable Scenario로 변환한다. 이후 실제 RF 장비는 Ground-truth Validation에 사용하고, 일상적인 Patch Regression은 Software-level Verification으로 자동화한다.**

---

# Lauren의 접근과 비교

Lauren:

```text
Human이 매번 UI 확인
       ↓
Playwright + CLI + Trace
       ↓
Agent가 반복 Verification
```

LTE / Modem 환경:

```text
Engineer가 매번 Shield Room Test
       ↓
Golden Trace + Scenario Model + Replay Harness
       ↓
Software가 반복 Verification
```

**본질적으로 같은 Engineering Approach**이다.

---

# 첫 번째 PoC 방향

현재 단계에서 가장 가치가 높은 첫 PoC는 모든 Mobility Scenario를 한 번에 모델링하는 것이 아니다.

하나의 Scenario를 골라 다음 Flow를 End-to-End로 한번 관통시켜 보는 것이 좋다.

```text
Real Equipment
      ↓
Golden Trace
      ↓
Input Extraction
      ↓
SW Replay
      ↓
Expected State / Output Comparison
```

이 흐름이 하나라도 성공하면 이후

```text
Handover
Cell Reselection
Reject
Downgrade
Security Detection
...
```

등을 같은 Framework로 확장할 수 있다.
