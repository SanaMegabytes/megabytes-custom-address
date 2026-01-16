
This document describes the **two security mechanisms** implemented to protect the network against abuse related specifically to **MNS name creation transactions** (`mns_register`).

These protections operate at the **policy** and **network** layers and are intentionally designed **without modifying consensus rules**.

---

## 1) Mempool Policy Protection (Standardness Rules)

### Purpose
Prevent attackers from spamming the network with malformed, oversized, or abusive MNS transactions, even if they are willing to pay transaction fees.

### Location
policy/policy.cpp  
Function: IsStandardTx(...)  
Loop: for (const CTxOut& txout : tx.vout)

### Mechanism
- MNS registrations are encoded using OP_RETURN (TxoutType::NULL_DATA).
- When such an output is detected:
  - The payload is strictly validated (format, size, and content).
  - Only **one OP_RETURN** is permitted per transaction.
  - Non-conforming MNS payloads are rejected.
- On validation failure:
```cpp
  reason = "mns-opreturn";
  return false;
 ```

### Effect
- The transaction is rejected by **mempool policy** and is not relayed by standard nodes.
- No consensus change is introduced.
- Miners could technically include such transactions, but default network behavior prevents their propagation.

### Rationale
This follows the Bitcoin / DigiByte design philosophy:  
**policy protects the network, consensus remains minimal**.

---

## 2) Network-Level Rate Limiting (MNS TX Flood Protection)

### Purpose
Mitigate denial-of-service attempts where a peer attempts to flood the network with large volumes of MNS creation transactions.

### Location
net_processing.cpp  
Handler: if (msg_type == NetMsgType::TX)

### Mechanism
- After deserializing an incoming transaction:
  - Detect whether it is an MNS transaction (for example, OP_RETURN containing the "MNS" marker).
- Maintain a per-peer rate counter within a rolling time window.
- If a peer exceeds the allowed MNS transaction rate:
  - The transaction is immediately dropped.
  - The peer is penalized using the standard misbehavior scoring system:
    
    ```cpp
    Misbehaving(*peer, 100, "mns-tx-rate");
    ```

### Enforcement Model
- The current configuration treats **ban as the norm**:
  - A high misbehavior score triggers automatic banning according to global node policy.
  - No custom ban logic is required.
  - Hard disconnect added :
  
```cpp
pfrom.fDisconnect = true;
 ```

---

## Combined Security Outcome

- **Mempool policy** filters abusive MNS payloads before they propagate.
- **Network rate limiting** protects peers from CPU, memory, and relay abuse.
- The cost of large-scale MNS spam becomes economically and technically prohibitive.
- Honest users remain unaffected.
- The network remains robust even if an attacker controls their own node.

---

## Design Philosophy

- No consensus changes.
- No special-case rules in block validation.
- Defense-in-depth using existing Bitcoin-style primitives.
- Predictable, auditable behavior aligned with upstream best practices.

---
