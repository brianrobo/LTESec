# LTE Security 접근법 — Rule 추가보다 원리와 Invariant 중심으로

방향은 거의 맞다.

다만 **“원리를 이해하고 그것을 차단한다”**를 조금 더 정확히 표현하면,

> **개별 공격 패턴을 끝없이 Rule로 추가하는 대신, 여러 공격이 공통으로 이용하는 프로토콜의 약점·비정상 상태 전이를 이해하고, 그 원리를 기준으로 탐지/차단 로직을 설계한다**

에 가깝다.

---

# 1. Rule-based로만 접근하면 생기는 문제

예를 들어 공격을 하나씩 Rule로 추가하면 이런 식이 된다.

```text
Rule 1: TAU Reject cause X가 N번 반복되면 공격 의심
Rule 2: Attach Reject cause Y가 N번 반복되면 공격 의심
Rule 3: 특정 RAT downgrade 후 특정 message가 오면 공격 의심
Rule 4: 특정 paging 패턴이면 위치 추적 의심
Rule 5: 특정 cipher 조합이면 공격 의심
...
Rule 100
Rule 200
```

문제는 공격자가 조금만 형태를 바꾸거나, 새로운 변형이 생기면 Rule이 계속 늘어난다는 점이다.

그리고 Rule이 많아질수록

- Rule끼리 충돌할 수 있고
- 정상망 동작을 공격으로 오인할 수 있고
- 예외 처리가 계속 늘어나고
- 유지보수가 어려워지고
- 새로운 변형 공격에 취약해질 수 있다.

즉,

> **Attack 하나가 늘어날 때마다 Detection Rule 하나가 늘어나는 구조는 확장성이 떨어진다.**

---

# 2. 원리 중심으로 보면 질문이 달라진다

단순히

> “이 공격의 메시지 패턴은 무엇인가?”

만 보는 것이 아니라,

다음 질문으로 넘어가야 한다.

```text
이 공격은 LTE의 어떤 trust assumption을 이용하는가?
        ↓
왜 UE가 이 상태로 유도될 수 있는가?
        ↓
정상 네트워크라면 어떤 상태 전이가 예상되는가?
        ↓
현재 관찰된 상태 전이가 정상 범위인가?
```

즉 공격 이름보다 먼저

> **이 공격이 왜 가능한가?**

를 본다.

---

# 3. 예: Downgrade 공격을 하나씩 보지 않는 방식

여러 종류의 downgrade 공격을 각각 별도 Rule로 관리할 수도 있다.

예:

```text
3G downgrade
2G downgrade
cipher downgrade
integrity capability downgrade
특정 reject 후 downgrade
```

이렇게 하면 각각 다른 공격으로 보이기 쉽다.

하지만 원리 기준으로 보면 이들을 하나의 상위 개념으로 묶을 수 있다.

예를 들면,

> **“현재 radio/network condition과 정상 mobility procedure로 설명되지 않는 강제적인 security capability degradation이 발생했는가?”**

라는 관점이다.

구조로 보면 다음과 같다.

```text
정상 상태
   ↓
Security capability / RAT level
   ↓
비정상적으로 약한 상태로 전이
   ↓
그 전이를 설명할 legitimate cause가 있는가?
```

즉 세부 공격 이름을 전부 외우기보다,

> **정상적으로 허용되는 상태 전이와 비정상적인 degradation을 구분하는 기준**

을 만드는 것이다.

---

# 4. 원리 기반이라고 Rule이 없어지는 것은 아니다

여기서 중요한 점이 있다.

> **원리 기반이라고 해서 Rule이 없어지는 것은 아니다.**

Rule의 구조가 바뀌는 것이다.

## 나쁜 접근

```text
Attack A → Rule A
Attack B → Rule B
Attack C → Rule C
Attack D → Rule D
Attack E → Rule E
```

공격 하나당 Rule 하나가 생긴다.

---

## 더 좋은 접근

```text
Protocol Principle 1
 ├─ Signal A
 ├─ Signal B
 └─ Signal C

Protocol Principle 2
 ├─ Signal D
 └─ Signal E
```

즉 Rule을

**attack name 기준**

으로 계속 늘리는 것이 아니라,

- Protocol invariant
- Security property
- State transition
- 정상/비정상 behavior

기준으로 상위 그룹화한다.

---

# 5. 핵심 개념: Invariant

LTE Security에 특히 잘 맞는 개념이 **Invariant**이다.

쉽게 말하면,

> **“정상망이라면 이런 조건은 유지되어야 한다.”**

라는 불변 조건이다.

예를 들어 개념적으로 다음과 같은 식이다.

- 충분한 Security Context가 만들어진 뒤에는 이유 없이 더 약한 cipher/integrity 상태로 가면 안 된다.
- 정상 mobility 절차에서는 특정 reject/redirect가 비정상적으로 반복되지 않아야 한다.
- Serving cell 상태와 Neighbor cell 측정값으로 설명되지 않는 이동이 반복되면 이상하다.
- 식별자 요청은 정상 procedure와 timing/context 안에서 발생해야 한다.

이런 식으로

**정상 시스템이 지켜야 하는 성질**

을 먼저 정의하는 것이다.

---

# 6. Attack Signature Matching보다 Invariant Violation Detection

기존 공격 탐지는 쉽게 이런 식이 된다.

```text
Attack signature matching
```

즉,

> “이 Pattern이면 Attack A”

라는 식이다.

하지만 원리 중심 접근은 다음에 가깝다.

```text
Protocol invariant violation detection
```

즉,

> **“이 동작이 특정 공격 이름과 정확히 일치하는가?”**

보다

> **“이 동작이 정상 프로토콜이 유지해야 하는 조건을 깨뜨렸는가?”**

를 본다.

---

# 7. LTE Security 공부 방향과 연결

이 접근이 LTE Security를 공부할 때 생각했던 방향과 잘 맞는다.

단순히

> **“어떤 공격이 존재하는가?”**

에서 끝나는 것이 아니라,

> **“LTE Protocol의 어떤 설계 특성 때문에 이 공격이 가능한가?”**

를 봐야 한다.

그리고 최종적으로

> **“UE / Modem은 어떤 정보를 이용해서 이 공격을 탐지할 수 있는가?”**

까지 연결해야 한다.

이를 한 단계 더 구조화하면 다음과 같다.

```text
Protocol design
    ↓
Security assumption
    ↓
Attack surface
    ↓
Observable abnormality
    ↓
Invariant / Detection logic
    ↓
Policy
```

---

# 8. 이 접근의 핵심은 Pattern이 아니라 원인이다

기존 방식은 이렇게 된다.

```text
공격 A
공격 B
공격 C
공격 D
공격 E
...
```

그리고 각각 Detection Rule을 만든다.

하지만 더 좋은 접근은 다음과 같다.

```text
왜 이 공격이 가능한가?
        ↓
어떤 Protocol assumption을 이용하는가?
        ↓
어떤 정상 상태를 깨뜨리는가?
        ↓
UE가 무엇을 관찰할 수 있는가?
        ↓
그 관찰값으로 Detection이 가능한가?
```

즉,

```text
공격 패턴을 외운다
        X
```

보다

```text
원리를 이해한다
        ↓
관찰 가능한 정보를 정의한다
        ↓
그 정보를 기반으로 탐지한다
        ↓
검증한다
```

라는 접근이다.

---

# 9. 차단은 가장 마지막 단계다

여기서 **차단(blocking)** 은 조금 더 조심해서 봐야 한다.

원리를 이해했다고 해서 바로 차단하는 것은 위험할 수 있다.

왜냐하면 정상망 동작까지 차단할 가능성이 있기 때문이다.

따라서 실제 상용화 관점에서는 보통 다음 순서가 더 안전하다.

```text
1. Detect
2. Collect evidence
3. Score confidence
4. Correlate repeated events
5. Mitigate
```

즉 처음부터

```text
Attack 감지
→ Cell block
```

으로 가는 것이 아니라,

```text
이상 현상 감지
    ↓
증거 수집
    ↓
반복성 / 연관성 분석
    ↓
Confidence 평가
    ↓
대응 정책 결정
```

으로 간다.

---

# 10. 현재 PoC처럼 Detection Only로 시작하는 이유

현재 PoC처럼 처음에는

```text
Detection only
```

로 가는 것이 자연스럽다.

먼저 실제 데이터를 수집한다.

예를 들어,

```text
어떤 유형이 얼마나 발생하는가?
어떤 Cell에서 반복되는가?
동일 이벤트가 얼마나 반복되는가?
복수 이벤트가 같이 발생하는가?
정상망에서도 비슷한 현상이 발생하는가?
```

이런 데이터를 본다.

그다음 False Positive를 충분히 확인한 뒤 대응 수준을 올린다.

예를 들면,

```text
low confidence
→ report only

medium confidence
→ priority down

high confidence
→ block / avoid cell
```

같은 방식이다.

---

# 11. 핵심 문장

한 문장으로 압축하면 다음과 같다.

> **Rule을 없애자는 것이 아니라, “공격 하나당 Rule 하나”에서 벗어나 여러 공격을 설명할 수 있는 프로토콜 원리와 불변조건을 먼저 정의하고, 그 원리를 위반하는 현상을 탐지하자는 접근이다.**

---

# 12. Lauren의 AI Agent 접근과 비슷한 이유

이 부분이 Lauren의 AI Coding Agent 접근과 상당히 비슷하다.

AI Agent에서 나쁜 접근은 다음과 같다.

```text
실수 하나
   ↓
Prompt Rule 하나 추가

또 다른 실수
   ↓
Prompt Rule 하나 추가

또 실수
   ↓
Prompt Rule 추가
```

즉,

```text
AI Agent:
실수 하나 → Prompt Rule 하나 추가
             X
```

이다.

더 좋은 접근은

```text
실패 원인 이해
    ↓
왜 실패했는지 분석
    ↓
무엇을 관찰하지 못했는지 확인
    ↓
반복되는 문제를
Test / Lint / Architecture / Skill로 일반화
```

하는 것이다.

즉,

```text
AI Agent:
Failure
   ↓
Root Cause
   ↓
Invariant / Test / Lint / Architecture
```

로 간다.

---

# 13. LTE Security도 같은 구조다

LTE Security도 똑같이 볼 수 있다.

```text
공격 하나 → Detection Rule 하나 추가
             X
```

대신,

```text
Protocol weakness 이해
        ↓
정상 상태 정의
        ↓
Observable signal 정의
        ↓
Invariant violation 탐지
```

로 간다.

---

# 14. 두 분야의 공통 구조

## AI Coding Agent

```text
Agent Failure
     ↓
Failure Cause
     ↓
Observable Signal
     ↓
Verification
     ↓
Correction
     ↓
System Improvement
```

## LTE Security

```text
Attack
     ↓
Protocol Weakness
     ↓
Observable Signal
     ↓
Detection
     ↓
Verification
     ↓
Mitigation
```

구조적으로 상당히 비슷하다.

---

# 15. 공통 철학

둘의 공통 철학은 결국 이것이다.

> **사례를 계속 외우지 말고, 사례들이 깨뜨리는 공통 원칙을 찾아라.**

그리고

> **그 공통 원칙을 시스템이 검사하게 만들어라.**

이렇게 하면 Rule 수를 줄이는 것 이상의 장점이 있다.

가장 중요한 장점은,

> **아직 이름이 붙지 않은 새로운 변형 공격도 탐지할 가능성이 생긴다.**

왜냐하면 특정 Attack Signature를 보는 것이 아니라,

**정상 시스템이 지켜야 하는 원칙이 깨졌는지**

를 보기 때문이다.

---

# 16. 가장 중요한 사고방식 변화

기존 질문:

> “이 Attack Pattern을 어떻게 잡지?”

보다

다음 질문이 더 중요하다.

> **“이 Attack이 깨뜨리는 정상 Protocol Property는 무엇이지?”**

그리고

> **“그 Property가 깨졌다는 것을 UE가 어떤 Signal로 관찰할 수 있지?”**

이다.

즉,

```text
Attack Pattern
     ↓
Rule
```

보다

```text
Protocol Principle
     ↓
Invariant
     ↓
Observable Signal
     ↓
Detection
```

으로 사고를 바꾸는 것이다.

---

# Final Takeaway

이 접근을 가장 짧게 표현하면 다음과 같다.

> **Rule-based를 버리는 것이 아니다. Rule의 기준을 공격 이름에서 Protocol Principle로 올리는 것이다.**

즉,

```text
Attack A → Rule A
Attack B → Rule B
Attack C → Rule C
```

가 아니라,

```text
Protocol Principle
      ↓
Invariant
      ↓
Multiple observable signals
      ↓
Detection
```

으로 만드는 것이다.

그리고 최종적으로는

> **“무슨 공격인가?”를 먼저 묻기보다, “정상 시스템의 어떤 원칙이 깨졌는가?”를 먼저 묻는 접근**

이라고 이해하면 가장 정확하다.
