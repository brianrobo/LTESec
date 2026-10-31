# LTE Security Research Approach

## Research Philosophy

LTE Security 연구의 목적은 단순히 새로운 공격 패턴을 계속 추가하는 것이 아니다.

핵심은 다음과 같은 접근이다.

> **“어떤 공격이 존재하는가?”**

에서 끝나는 것이 아니라,

> **“LTE protocol의 어떤 설계 특성 때문에 이 공격이 가능한가?”**

를 이해하고,

최종적으로는

> **“UE / Modem은 어떤 정보를 이용해 이 공격을 탐지할 수 있는가?”**

그리고 더 나아가,

> **“탐지 이후 어떤 대응책을 마련할 수 있는가?”**

까지 연결하는 것을 목표로 한다.

---

# 1. 올바른 접근 방향

단순히 공격 패턴을 계속 추가하는 방식은 한계가 있다.

예를 들어 다음과 같은 접근은 시간이 지날수록 관리하기 어려워진다.

```text
Attack Pattern A
Attack Pattern B
Attack Pattern C
Attack Pattern D
Attack Pattern E
...
```

이 방식은 결국 다음과 같은 Signature 기반 탐지로 귀결되기 쉽다.

```text
A 메시지 발생 → 공격 의심

B Cause 발생 → 공격 의심

C Sequence 발생 → 공격 의심
```

문제는 공격자가 메시지 조합, 순서, Cause, RF 조건 등을 조금만 변경해도 새로운 패턴처럼 보일 수 있다는 점이다.

따라서 중요한 것은 **공격의 형태를 외우는 것이 아니라 공격이 성립하는 원리를 이해하는 것**이다.

---

# 2. Principle-Based Security Analysis

LTE Security를 분석할 때는 다음 흐름으로 접근한다.

```text
Normal Protocol Behavior
        ↓
Security Assumption
        ↓
Trust Boundary
        ↓
Protection Mechanism
        ↓
Protection Gap / Exception
        ↓
Attack Surface
        ↓
Attack Technique
        ↓
Protocol / UE Impact
        ↓
Observable Evidence
        ↓
Detection
        ↓
Mitigation
```

즉 공격 자체보다 먼저 다음 질문을 던진다.

---

## Step 1. 정상 동작은 무엇인가?

먼저 3GPP 기준 정상 protocol flow를 이해한다.

예:

```text
UE
 │
 │ TAU Request
 ↓
MME
 │
 │ TAU Accept
 ↓
UE
```

그리고 정상 상태에서는

- 어떤 state에서 message가 전송되는지
- 어떤 timer가 동작하는지
- 어떤 security context가 존재하는지
- 어떤 message가 integrity protected 되는지

를 확인한다.

---

# 3. Security Assumption을 찾는다

다음 단계는 LTE protocol이 어떤 것을 신뢰하도록 설계되었는지 파악하는 것이다.

예를 들어 다음 질문을 한다.

```text
UE는 이 메시지를 왜 신뢰하는가?

이 메시지는 authentication 이후에 오는가?

Integrity protection이 적용되는가?

Ciphering이 적용되는가?

Serving network identity가 검증된 상태인가?

UE state를 변경할 수 있는 메시지인가?
```

LTE attack의 상당수는 결국 이러한 **security assumption 또는 trust boundary**와 관련된다.

---

# 4. Protection이 적용되지 않는 구간을 찾는다

모든 LTE signalling이 처음부터 보호되는 것은 아니다.

대표적인 예:

```text
Initial Access

Identity Procedure

Authentication 이전 Signalling

Security Mode 이전 Signalling

Broadcast Information

Paging
```

따라서 중요한 질문은 다음과 같다.

> **“이 메시지가 UE의 상태를 변경할 수 있는데 아직 cryptographic protection은 없는가?”**

이 지점이 중요한 Attack Surface가 될 수 있다.

---

# 5. Attack Surface를 정의한다

Attack Surface는 단순히 특정 공격 이름을 의미하지 않는다.

예를 들어 다음과 같은 protocol property 자체가 Attack Surface가 될 수 있다.

```text
Unauthenticated Message

Unprotected Broadcast Information

Pre-Authentication Reject

Temporary Identifier Exposure

User Plane Integrity Absence

Legacy RAT Interworking

Cell Selection / Reselection
```

---

# 6. 하나의 원리에서 여러 공격이 파생될 수 있다

예를 들어 다음 property를 생각해본다.

```text
Pre-Authentication Signalling
```

이 하나의 특성에서 여러 공격이 파생될 수 있다.

```text
Pre-Authentication Signalling
        │
        ├── Identity Request
        │      └── IMSI Exposure
        │
        ├── Attach Reject
        │      └── DoS
        │
        ├── TAU Reject
        │      └── Mobility / Service DoS
        │
        ├── Service Reject
        │      └── Service Disruption
        │
        └── Authentication Reject
               └── Registration Failure
```

따라서 공격 이름을 각각 독립적으로 외우는 것보다

```text
Pre-Authentication Trust
```

라는 공통 원리를 먼저 이해하는 것이 중요하다.

---

# 7. 예시: TAU Reject Attack

단순한 공격 패턴 중심 접근은 다음과 같다.

```text
TAU Reject가 오면 공격 가능
```

하지만 이 정도로는 충분하지 않다.

보다 중요한 질문은 다음과 같다.

```text
Why can TAU Reject affect UE state?

↓   

Does UE accept it before authentication?

↓

Is integrity protection required?

↓

Which EMM Cause values change UE behavior?

↓

Which timer starts?

↓

How long does the UE remain affected?

↓

What recovery condition exists?

↓

Can an attacker reproduce the required condition?
```

즉 핵심은

```text
TAU Reject
```

라는 공격 이름이 아니라,

> **신뢰 관계가 완전히 형성되기 이전의 signalling이 UE의 protocol state를 변경할 수 있다**

라는 설계 특성이다.

이 원리를 이해하면 공격자가

- Cause를 바꾸거나
- Sequence를 바꾸거나
- Timer 조건을 바꾸거나
- Cell 환경을 바꾸더라도

공격의 본질을 파악할 수 있다.

---

# 8. Signature Detection의 한계

다음과 같은 탐지는 쉽게 구현할 수 있다.

```text
TAU Reject Cause = X
        ↓
Threat Detected
```

그러나 정상 network에서도 동일한 Cause가 발생할 수 있다.

따라서 단일 protocol event만으로 공격을 판단하면 False Positive가 증가한다.

보다 나은 접근은 여러 context를 함께 보는 것이다.

```text
Protocol Context
+
Radio Context
+
Cell Context
+
Security Context
+
Temporal Behavior
```

---

# 9. Context-Based Detection

예를 들어 Fake Cell Detection을 생각한다.

단순히 다음 조건만 사용하면 충분하지 않다.

```text
Strong RSRP
      ↓
Fake Cell
```

정상 network에서도 강한 RSRP는 흔히 발생할 수 있기 때문이다.

대신 여러 evidence를 조합한다.

```text
Sudden Cell Selection

        +

Unexpected TAC Change

        +

Unexpected PLMN Behavior

        +

RAT Downgrade

        +

Abnormal Reject Sequence

        +

Serving / Neighbor RF Difference

        +

Previous Camping History
```

이러한 정보를 함께 분석하면 훨씬 의미 있는 detection logic을 만들 수 있다.

즉 Detection은

```text
Single Signature
```

보다

```text
Multi-Context Evidence
```

기반으로 접근하는 것이 바람직하다.

---

# 10. Detection에 활용 가능한 Context

## Protocol Context

```text
NAS Message

RRC Message

Security Header

EMM Cause

Timer

Security Context

Security Algorithm

UE Security Capability
```

---

## Radio Context

```text
RSRP

RSRQ

SINR

EARFCN

PCI

Serving Cell

Neighbor Cell
```

---

## Mobility Context

```text
Cell Change

TAC Change

PLMN Change

RAT Change

Reselection History

Handover History
```

---

## Temporal Context

```text
Repeated Reject

Repeated Identity Request

Repeated Registration Failure

Short-Time RAT Change

Repeated Cell Reselection

Periodic Abnormal Event
```

---

# 11. Detection 연구의 핵심

단순히 다음을 구현하는 것이 목적이 아니다.

```text
Attack A → Alarm A

Attack B → Alarm B

Attack C → Alarm C
```

보다 발전된 형태는 다음과 같다.

```text
Evidence
   ↓
Abnormal Behavior
   ↓
Security Principle Violation
   ↓
Threat Classification
```

---

# 12. Modem Security Event로 연결

최종적으로는 protocol 연구 결과가 실제 modem detection 구조로 연결되어야 한다.

예:

```text
Protocol Principle

        ↓

Violation / Abnormal Behavior

        ↓

Evidence

        ↓

Threat Category

        ↓

Reason Code

        ↓

Confidence / Threat Score

        ↓

Security Event Report

        ↓

Mitigation Policy
```

---

# 13. Threat Category와 Evidence 분리

특히 중요한 설계 원칙은

```text
Threat
```

와

```text
Evidence
```

를 구분하는 것이다.

예:

```text
Threat

RAT Downgrade
```

하지만 Evidence는 여러 개일 수 있다.

```text
Unexpected RAT Change

Security Capability Mismatch

Cell Reselection Behavior

Neighbor Cell Condition

Reject Sequence

Serving Cell Signal
```

따라서 구조적으로는 다음과 같이 보는 것이 좋다.

```text
Multiple Evidence
       ↓
Threat Classification
```

이렇게 하면 새로운 공격 변형이 발생해도 기존 evidence 조합을 통해 탐지할 가능성이 높아진다.

---

# 14. False Positive까지 연구 대상에 포함한다

Security detection에서는 공격 탐지만큼 중요한 것이 False Positive다.

따라서 각 공격을 연구할 때 반드시 다음 항목을 포함한다.

```text
Normal Scenario that looks like an Attack

False Positive Condition

False Negative Condition

Required Correlation

Minimum Evidence

Confidence Level
```

예:

```text
RAT Downgrade
```

가 발생했다고 무조건 공격은 아니다.

정상적인 경우:

```text
LTE Coverage Loss

Network Congestion

Operator Mobility Policy

Emergency Condition

Weak RF Environment
```

에서도 RAT downgrade가 발생할 수 있다.

따라서 단순히

```text
LTE → WCDMA
```

만으로 공격을 판단할 수 없다.

---

# 15. 최종 연구 Template

앞으로 모든 LTE Security 주제를 다음 형식으로 분석한다.

```text
1. Normal Operation

2. 3GPP Specification

3. Protocol State

4. Security Assumption

5. Trust Boundary

6. Protection Mechanism

7. Protection Gap

8. Attack Surface

9. Attack Prerequisite

10. Attack Procedure

11. Attack Variants

12. Protocol Impact

13. UE State Impact

14. Network Impact

15. Observable Evidence

16. Modem Observable Data

17. AP Observable Data

18. Detection Logic

19. False Positive Conditions

20. False Negative Conditions

21. Threat Classification

22. Confidence / Scoring

23. Mitigation

24. Representative Papers

25. Existing PoC / GitHub

26. Remaining Research Questions
```

---

# 16. Research Direction

앞으로 LTE Security 연구는 다음과 같은 방향으로 진행한다.

```text
Attack Collection
       ↓
Attack Classification
       ↓
Protocol Principle Analysis
       ↓
Root Cause Analysis
       ↓
Observable Evidence
       ↓
Detection Logic
       ↓
False Positive Analysis
       ↓
Threat Scoring
       ↓
Mitigation
```

---

# 17. Research Goal

최종 목표는 단순한 LTE 공격 백과사전을 만드는 것이 아니다.

목표는 다음 질문에 답할 수 있는 **LTE Security Engineering Knowledge Base**를 만드는 것이다.

### Question 1

> 어떤 공격이 존재하는가?

### Question 2

> 왜 이 공격이 LTE protocol에서 가능한가?

### Question 3

> 어떤 security assumption 또는 trust boundary가 관련되어 있는가?

### Question 4

> UE / Modem에서 어떤 evidence를 관찰할 수 있는가?

### Question 5

> 정상 network behavior와 어떻게 구분할 것인가?

### Question 6

> 어떤 detection logic이 가능한가?

### Question 7

> Detection
