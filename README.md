# LTE Security Research Roadmap

> LTE Security를 **Architecture → Protocol Security → Attack Surface → Attack Techniques → Research → Tools → Detection & Mitigation** 순으로 체계적으로 연구하기 위한 개인 학습 및 연구 자료 모음.

---

## Table of Contents

- [1. LTE Security 전체 구조](#1-lte-security-전체-구조)
- [2. 추천 학습 순서](#2-추천-학습-순서)
- [3. Standards](#3-standards)
- [4. Books & eBooks](#4-books--ebooks)
- [5. LTE Security Architecture](#5-lte-security-architecture)
- [6. Authentication & Key Management](#6-authentication--key-management)
- [7. NAS Security](#7-nas-security)
- [8. AS / RRC / PDCP Security](#8-as--rrc--pdcp-security)
- [9. Identity & Privacy](#9-identity--privacy)
- [10. Pre-Authentication Attack Surface](#10-pre-authentication-attack-surface)
- [11. Fake Base Station / IMSI Catcher](#11-fake-base-station--imsi-catcher)
- [12. Downgrade Attacks](#12-downgrade-attacks)
- [13. DoS / Availability Attacks](#13-dos--availability-attacks)
- [14. Layer 2 / User Plane Attacks](#14-layer-2--user-plane-attacks)
- [15. Protocol Fuzzing & Implementation Security](#15-protocol-fuzzing--implementation-security)
- [16. Detection & Mitigation](#16-detection--mitigation)
- [17. GitHub / Open Source Tools](#17-github--open-source-tools)
- [18. Important Research Papers](#18-important-research-papers)
- [19. YouTube / Lectures](#19-youtube--lectures)
- [20. Coursera / Online Courses](#20-coursera--online-courses)
- [21. Research Roadmap](#21-research-roadmap)
- [22. Recommended Repository Structure](#22-recommended-repository-structure)

---

# 1. LTE Security 전체 구조

```text
LTE SECURITY
│
├── 1. Security Architecture / Trust Model
│   ├── UE
│   ├── USIM
│   ├── eNB
│   ├── MME
│   ├── HSS / AuC
│   └── EPC Interfaces
│
├── 2. Authentication & Key Management
│   ├── EPS-AKA
│   ├── RAND / AUTN
│   ├── RES / XRES
│   ├── CK / IK
│   ├── KASME
│   ├── NAS Keys
│   └── AS Keys
│
├── 3. NAS Security
│   ├── NAS Security Context
│   ├── Security Mode Command
│   ├── EEA / EIA selection
│   ├── KNASenc
│   ├── KNASint
│   ├── NAS COUNT
│   ├── Replay Protection
│   ├── Integrity Protection
│   └── Ciphering
│
├── 4. AS / RRC / PDCP Security
│   ├── KeNB
│   ├── KRRCenc
│   ├── KRRCint
│   ├── KUPenc
│   ├── Security Mode Procedure
│   └── Handover Key Derivation
│
├── 5. Identity & Privacy
│   ├── IMSI
│   ├── GUTI
│   ├── S-TMSI
│   ├── IMSI Catcher
│   ├── Paging Tracking
│   └── Location Tracking
│
├── 6. Pre-Authentication Attack Surface
│   ├── Identity Request
│   ├── Attach Reject
│   ├── TAU Reject
│   ├── Service Reject
│   ├── Authentication Reject
│   ├── Security Capability Manipulation
│   └── Rogue eNB
│
├── 7. Downgrade / Fake Cell / Cell Selection
│   ├── LTE → 3G Downgrade
│   ├── LTE → 2G Downgrade
│   ├── Fake Base Station
│   ├── Reselection Manipulation
│   ├── SIB Manipulation
│   └── Signal Overshadowing
│
├── 8. DoS / Availability
│   ├── Reject-based DoS
│   ├── Paging Storm
│   ├── Attach Storm
│   ├── Jamming
│   └── Malformed Signalling
│
├── 9. Layer 2 / User Plane Attacks
│   ├── RNTI Tracking
│   ├── Website Fingerprinting
│   ├── ALTER
│   ├── IMP4GT
│   └── User Plane Integrity
│
├── 10. Implementation Security
│   ├── Malformed NAS
│   ├── Malformed RRC
│   ├── Protocol Fuzzing
│   ├── Negative Testing
│   ├── State Machine Bugs
│   └── Baseband Vulnerabilities
│
└── 11. Detection / Mitigation
    ├── Rogue Cell Detection
    ├── Behavioral Detection
    ├── Anomaly Detection
    ├── Threat Scoring
    ├── UE / Modem Reporting
    └── LTE → 5G Security Evolution
```

---

# 2. 추천 학습 순서

LTE Security를 처음부터 논문 중심으로 접근하면 전체 구조를 놓치기 쉽다.

추천 순서는 다음과 같다.

```text
LTE Architecture
       ↓
EPS Authentication
       ↓
Key Hierarchy
       ↓
NAS Security
       ↓
AS Security
       ↓
Pre-authentication Signalling
       ↓
Identity / Privacy
       ↓
Fake Base Station
       ↓
Downgrade
       ↓
Reject / DoS
       ↓
Paging / Location Tracking
       ↓
Layer 2 Attacks
       ↓
Protocol Fuzzing
       ↓
Detection / Mitigation
```

특히 **NAS Protocol / Modem Security 관점**에서는 다음 순서를 우선한다.

```text
NAS Security
   ↓
Pre-authentication Attack Surface
   ↓
Identity / Privacy
   ↓
Reject / DoS
   ↓
Downgrade
   ↓
Fake Cell
   ↓
Location Tracking
   ↓
Detection & Mitigation
```

---

# 3. Standards

## 3.1 3GPP TS 33.401

**3GPP TS 33.401 — System Architecture Evolution (SAE); Security Architecture**

LTE Security의 가장 중요한 specification.

### 주요 연구 항목

- EPS-AKA
- Security Context
- Key Hierarchy
- KASME
- KNASenc
- KNASint
- KeNB
- NAS Integrity Protection
- NAS Ciphering
- AS Security
- Algorithm Negotiation
- Handover Security

### Reference

- 3GPP Specification  
  https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=2296

### Priority

**★★★★★**

---

## 3.2 3GPP TS 24.301

**Non-Access-Stratum (NAS) protocol for Evolved Packet System**

NAS protocol 및 EMM/ESM state machine을 분석할 때 핵심 specification.

### 중요 메시지

```text
Attach Request
Attach Accept
Attach Reject

TAU Request
TAU Accept
TAU Reject

Service Request
Service Reject

Identity Request
Identity Response

Authentication Request
Authentication Response
Authentication Reject

Security Mode Command
Security Mode Complete
Security Mode Reject
```

### 중요 Timer / Cause

```text
EMM Cause

T3245
T3346
T3402
T3411
```

Security attack은 결국 다음 구조로 분석한다.

```text
NAS Message
    ↓
Security Protection 여부
    ↓
Current EMM State
    ↓
Message Acceptance Condition
    ↓
State Transition
    ↓
Timer / Cause
    ↓
Security Impact
```

### Priority

**★★★★★**

---

# 4. Books & eBooks

## 4.1 NIST SP 800-187

### Guide to LTE Security

무료 LTE Security 자료 중 가장 먼저 읽기 좋은 문서.

LTE architecture에서 authentication, key hierarchy, threats, mitigation까지 전반적인 구조를 설명한다.

### 주요 내용

- LTE Security Architecture
- EPS Authentication
- Key Management
- Confidentiality
- Integrity
- Network Security
- Threat Model
- Security Recommendations

### Link

https://csrc.nist.gov/pubs/sp/800/187/final

### Priority

**★★★★★**

---

## 4.2 LTE Security, 2nd Edition

Authors:

- Dan Forsberg
- Günther Horn
- Wolf-Dietrich Moeller
- Valtteri Niemi

LTE Security를 설계 철학까지 포함해서 이해하기 좋은 책.

### 중요 Chapter

```text
EPS Security Architecture

EPS Authentication and Key Agreement

EPS Protection for Signalling and User Data

Security in LTE Mobility

Cryptographic Algorithms

EPS / 3G / GSM Interworking Security
```

### Publisher

Wiley

https://www.wiley.com/en-us/lte-security-2nd-edition-p-9781118380659

### Priority

**★★★★★**

---

# 5. LTE Security Architecture

LTE Security를 분석할 때 먼저 Trust Boundary를 파악한다.

```text
     USIM
       │
       │
      UE
       │
       │ Uu
       │
      eNB
       │
       │ S1
       │
      MME
       │
       │
     HSS / AuC
```

각 구간의 보안 역할이 다르다.

### 주요 질문

- 누가 누구를 인증하는가?
- 어떤 구간이 ciphering 되는가?
- 어떤 구간이 integrity protected 되는가?
- 언제 security context가 생성되는가?
- 어떤 signalling이 authentication 이전에 허용되는가?

마지막 질문이 LTE 공격 연구에서 특히 중요하다.

---

# 6. Authentication & Key Management

LTE authentication 핵심은 **EPS-AKA**이다.

기본 흐름:

```text
UE                   MME                  HSS/AuC
│                     │                     │
│                     │ Authentication Data │
│                     ├────────────────────>│
│                     │                     │
│                     │ RAND/AUTN/XRES      │
│                     │<────────────────────┤
│                     │                     │
│ Authentication Req. │
│<────────────────────┤
│                     │
│ Authentication Resp │
├────────────────────>│
```

주요 값:

```text
RAND
AUTN
RES
XRES
CK
IK
KASME
```

---

## Key Hierarchy

```text
USIM Secret Key (K)
       │
       ↓
      AKA
       │
   CK / IK
       │
       ↓
    KASME
       │
       ├── KNASenc
       │
       ├── KNASint
       │
       └── KeNB
             │
             ├── KRRCenc
             ├── KRRCint
             └── KUPenc
```

이 구조는 LTE Security를 이해하는 핵심이다.

---

# 7. NAS Security

NAS Security는 UE ↔ MME signalling을 보호한다.

주요 security property:

```text
Confidentiality
Integrity
Replay Protection
Message Authentication
```

---

## NAS Security Context

연구할 항목:

- Native EPS Security Context
- Mapped Security Context
- NAS COUNT
- KSI
- Selected Ciphering Algorithm
- Selected Integrity Algorithm

---

## NAS Security Mode Command

```text
MME
 │
 │ Security Mode Command
 ↓
UE
 │
 │ Security Mode Complete
 ↓
MME
```

이 절차에서 중요하게 봐야 할 것:

- UE Security Capability
- Selected NAS Ciphering Algorithm
- Selected NAS Integrity Algorithm
- Replay Protection
- Downgrade Prevention

---

# 8. AS / RRC / PDCP Security

Access Stratum Security는 UE ↔ eNB 구간을 보호한다.

주요 key:

```text
KeNB

KRRCenc
KRRCint

KUPenc
```

관련 계층:

```text
RRC
PDCP
```

연구 포인트:

- AS Security Mode
- RRC Integrity Protection
- RRC Ciphering
- User Plane Ciphering
- User Plane Integrity 부재
- Handover Key Derivation

---

# 9. Identity & Privacy

LTE Security에서 중요한 연구 분야.

LTE에는 여러 temporary identifier가 존재한다.

```text
IMSI
GUTI
S-TMSI
C-RNTI
```

공격자는 identifier 간 mapping을 시도할 수 있다.

---

## 주요 공격

### IMSI Catching

```text
Fake eNB
    ↓
Identity Request
    ↓
UE
    ↓
IMSI Response
```

---

## Paging Tracking

Paging message를 이용하여 특정 UE의 위치 또는 존재 여부를 추적할 수 있다.

연구 포인트:

```text
Paging
S-TMSI
Temporary Identifier
Location Tracking
Presence Detection
```

---

# 10. Pre-Authentication Attack Surface

LTE Security에서 매우 중요한 공격 영역.

Authentication 이전에는 일부 signalling에 cryptographic protection이 존재하지 않는다.

대표적으로 연구해야 할 메시지:

```text
Identity Request

Attach Reject

TAU Reject

Service Reject

Authentication Reject
```

핵심 질문:

> Authentication 이전에 UE가 어떤 메시지를 신뢰하는가?

---

## 연구 구조

각 메시지를 다음 구조로 분석한다.

```text
Message
   ↓
Protected / Unprotected
   ↓
Acceptance Condition
   ↓
UE State Change
   ↓
Timer Start
   ↓
Network Availability Impact
   ↓
Possible Attack
```

---

# 11. Fake Base Station / IMSI Catcher

Fake eNB는 LTE security 연구의 핵심 공격 모델 중 하나다.

```text
Attacker
   │
Fake eNB
   │
   ↓
UE
```

공격자가 legitimate network보다 높은 cell selection priority 또는 강한 RF signal을 제공하면 UE가 rogue cell로 이동할 가능성이 있다.

연구 항목:

- Cell Selection
- Cell Reselection
- SIB manipulation
- PLMN information
- Tracking Area
- Signal strength
- Cell barring
- RAT priority

---

# 12. Downgrade Attacks

목표:

```text
LTE
 ↓
3G
 ↓
2G
```

낮은 RAT로 UE를 유도한 뒤 구형 network의 보안 취약점을 이용한다.

연구 항목:

- UE Security Capability
- RAT capability
- Cell reselection
- Inter-RAT mobility
- Authentication
- Algorithm negotiation
- Security Mode procedure

---

# 13. DoS / Availability Attacks

LTE에서는 signalling manipulation만으로 UE의 network availability에 영향을 줄 수 있다.

대표 공격:

```text
Attach Reject

TAU Reject

Service Reject

Authentication Reject

Paging Storm

Attach Storm

Jamming
```

---

## Reject-based DoS

```text
Fake eNB
   ↓
Attach Reject
   ↓
UE
   ↓
State Change
   ↓
Timer
   ↓
Network unavailable
```

분석 시 다음을 확인한다.

```text
EMM Cause
Timer
UE State
PLMN State
RAT State
Recovery Condition
```

---

# 14. Layer 2 / User Plane Attacks

LTE Security 연구는 NAS / RRC뿐 아니라 L2까지 확대되었다.

대표 연구:

- RNTI Tracking
- Website Fingerprinting
- ALTER
- IMP4GT
- User Plane Integrity

---

## Breaking LTE on Layer Two

LTE Layer 2의 metadata와 user-plane protection 한계를 이용한 공격 연구.

주요 공격:

```text
Identity Mapping

Website Fingerprinting

ALTER
```

ALTER 공격은 LTE user-plane의 integrity protection 부재를 이용한다.

---

# 15. Protocol Fuzzing & Implementation Security

Specification이 안전해도 modem implementation이 안전하다는 의미는 아니다.

연구해야 할 test input:

```text
Malformed Message

Invalid IE

Missing IE

Unexpected IE

Incorrect Message Order

Unexpected Message

Invalid Cause

Replay

Wrong Security Header

Invalid Integrity

Unexpected State Transition
```

---

## Negative Testing

정상 flow:

```text
Expected Message
       ↓
Expected State
```

Security testing:

```text
Unexpected Message
       ↓
Unexpected State?
       ↓
Crash?
       ↓
DoS?
       ↓
Memory Corruption?
```

---

# 16. Detection & Mitigation

공격 연구 다음 단계는 Detection이다.

연구 방향:

```text
Protocol Rule Detection

State Machine Detection

Cell Behavior Detection

Signal-based Detection

Statistical Detection

Machine Learning Detection

Cross-layer Detection
```

---

## Modem에서 관찰 가능한 정보

예:

```text
Serving Cell

PCI

EARFCN

PLMN

TAC

Cell ID

RSRP

RSRQ

SINR

RAT

NAS Message

Reject Cause

Security Algorithm

Security Context

UE Security Capability

Mobility History
```

이 정보를 이용해 공격 이벤트를 판단할 수 있다.

---

# 17. GitHub / Open Source Tools

## 17.1 Cellular Security Papers

Cellular Security 연구 논문을 정리한 repository.

### 포함 분야

- LTE
- 5G
- Formal Verification
- Protocol Fuzzing
- Fake Base Station
- Privacy
- DoS
- Baseband Security

### GitHub

https://github.com/onehouwong/Cellular-Security-Papers

### Priority

**★★★★★**

---

## 17.2 Awesome Cellular Hacking

Cellular hacking 관련 tool / paper / presentation 모음.

### 주요 주제

```text
LTE Public Warning System

Signal Overshadowing

Breaking LTE on Layer Two

LTE Protocol Exploits

Paging Storm

Fake Base Station

Baseband Attacks
```

### GitHub

https://github.com/W00t3k/Awesome-Cellular-Hacking

### Priority

**★★★★★**

---

## 17.3 srsRAN 4G

Open Source LTE Stack.

구성:

```text
srsUE

srsENB

srsEPC
```

연구 가능 계층:

```text
NAS
RRC
PDCP
RLC
MAC
PHY
S1AP
```

지원되는 security 기능에는 LTE authentication과 cipher/integrity algorithm 구현 등이 포함된다.

### GitHub

https://github.com/srsran/srsRAN_4G

### Priority

**★★★★★**

---

## 17.4 Open5GS

Open Source EPC / 5GC implementation.

LTE Core 구성:

```text
MME

HSS

SGW

PGW

PCRF
```

LTE NAS security를 network side에서 연구하기 좋다.

### GitHub

https://github.com/open5gs/open5gs

### Priority

**★★★★★**

---

## 17.5 pycrate

Python protocol parsing framework.

LTE / 5G NAS parser를 제공한다.

주요 module:

```text
NAS

NASLTE

NAS5G
```

활
