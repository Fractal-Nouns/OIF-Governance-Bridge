# 🏛️ Nounish Multi-Chain Governance Bridge  
### Layer 0-based Cross-Chain Messaging w/ Expiring-Block Trigger & Open Intents Integration

---

## 🔍 Overview

The **Nounish Multi-Chain Governance Bridge** enables autonomous, verifiable synchronization of governance state (proposals, votes, executions) across multiple EVM-compatible chains.

It combines:
- **Layer 0 cross-chain messaging** for trust-minimized communication  
- **Expiring-block gating** for deterministic timing  
- **On-chain DAO Relayer/Solver** to trigger execution  
- **Open Intents Framework (OIF / ERC-7683)** for standardized automation and solver incentives  

All components operate **fully on-chain** — no centralized servers or off-chain schedulers.

---

## 🧱 System Architecture

GovernanceRoot (Chain A)  
 │  
 ├── queues message (payload + expiryBlock)  
 │  
 ▼  
PoppingContract (Chain A)  
 │  
 ├── popMessage() callable after expiry  
 │  │  
 │  ▼  
 │ Layer0Endpoint (Chain A)  
 │  │  
 │  ▼  
 │ Layer0 Relay / Proof Transmission  
 │  │  
 │  ▼  
 ▼  
Layer0Endpoint (Chain B)  
 │  
 ▼  
BridgeReceiver (Chain B)  
 │  
 ▼  
GovernanceMirror / Execution  

A **DAO Relayer/Solver** monitors Chain A for expired messages and calls `popMessage()` once eligible.

---

## ⚙️ Components

### 1. GovernanceRoot (Chain A)

**Purpose:** Origin of proposals and votes. Queues cross-chain governance messages.

**Functions:**
- queueMessage(payload, durationBlocks)  
  - Creates a Message struct containing:  
    - payload: Encoded governance result  
    - expiryBlock: block.number + durationBlocks  
    - popped = false  
  - Emits MessageQueued(id, expiryBlock, payload)

All proposal data and timing are on-chain and auditable.

---

### 2. PoppingContract (Chain A)

**Purpose:** Handles message expiry and dispatch through Layer 0.

**Behavior:**
- Validates that the expiry block has passed  
- Marks the message as popped  
- Sends the payload through the Layer 0 endpoint  

This is the **on-chain trigger** that initiates the cross-chain message.

---

### 3. Layer0Endpoint (Chain A → Chain B)

**Purpose:** On-chain contract provided by the Layer 0 messaging protocol (LayerZero, Axelar, Hyperlane, etc.)

**Behavior:**
- Receives encoded payload via sendMessage()  
- Emits message event and proof metadata  
- Verifies proof and relays message to destination chain  

All proofs, messages, and receipts are **on-chain**.

---

### 4. DAO Relayer / Solver

**Purpose:** Executes the popMessage() call when conditions are met.

**Roles:**
- DAO-run Relayer (authorized executor)  
- Open Intents Solver (ERC-7683-compliant, permissionless actor)  

**Behavior:**
1. Monitors Chain A for MessageQueued events  
2. Detects when block.number ≥ expiryBlock  
3. Calls popMessage(id) on PoppingContract  
4. Optionally receives a reward for execution  

**Implementation Options:**
- Direct DAO-owned relayer  
- Intent-based solver marketplace using OIF standards  


For a more detailed breakdown of the Solver visit 
[Open Intent Solver](https://github.com/BaD-DAO/nouns-bridge/issues/9)

---

### 5. Layer0Endpoint (Chain B)

**Purpose:** Receives and validates cross-chain messages.  

**Behavior:**
- Verifies proofs from Chain A  
- Forwards validated payload to BridgeReceiver  

This endpoint is deployed by the Layer 0 protocol and operates fully on-chain.

---

### 6. BridgeReceiver / GovernanceMirror (Chain B)

**Purpose:** On-chain destination contract that processes received governance messages.

**Behavior:**
- Decodes payload (proposalId, outcome, etc.)  
- Executes governance action or updates mirror state  
- Emits GovernanceUpdated(proposalId, passed)

Ensures cross-chain governance results are synchronized and verifiable.

---

### 7. IntentManager (Optional, for Open Intents Integration)

**Purpose:** Registers and tracks ERC-7683-compliant intents representing cross-chain governance actions.

**Behavior:**
- Creates an intent when queueMessage() is called  
- Marks intent as Fulfilled once the corresponding message is processed  
- Allows solver rewards and DAO incentives  

**Events:**
- IntentCreated(intentId, sourceChain, destChain, expiryBlock, payload)  
- IntentFulfilled(intentId, txHash)

---

## 🔁 Message Lifecycle

| Step | Chain | Component | Description |
|------|--------|------------|-------------|
| 1 | A | GovernanceRoot | Proposal finalized; message queued (payload, expiryBlock) |
| 2 | A | IntentManager (opt.) | Intent registered |
| 3 | A | DAO Relayer / Solver | Detects expiry; calls popMessage(id) |
| 4 | A | PoppingContract | Validates expiry; sends via Layer0Endpoint |
| 5 | A | Layer0Endpoint | Emits event + proof for relay |
| 6 | B | Layer0Endpoint | Validates proof; forwards payload |
| 7 | B | BridgeReceiver | Executes governance update |
| 8 | B | IntentManager (opt.) | Marks intent fulfilled |

All transactions and events occur **on-chain**.

---

## 🧩 Example Payload Structure

payload:  
- proposalId  
- passed (boolean)  
- proposer (address)  
- executionData (bytes)

Encoded as bytes for transport through Layer 0 messaging.

---

## 🧠 Security and Design Considerations

- **Deterministic Expiry:** Uses block.number for on-chain timing  
- **Replay Protection:** Each message is marked popped once executed  
- **Finality Buffer:** Expiry block accounts for chain finality differences  
- **Proof Verification:** Handled by Layer 0 endpoint contracts  
- **Solver Incentives:** DAO may fund a pool for solver gas reimbursement  
- **Governance Autonomy:** DAO can upgrade endpoints and relayers via proposals  
- **Auditability:** All steps emit on-chain events  

---

## 🧮 Tech Stack

| Layer | Technology |
|-------|-------------|
| Smart Contracts | Solidity (EVM) |
| Messaging | Layer 0 (LayerZero / Axelar / Hyperlane) |
| Automation | DAO Relayer / OIF Solver |
| Governance | Nounish (GovernorBravo-style) |
| Verification | On-chain proofs via Layer 0 |

---

## 🧩 Future Extensions

- Multi-destination propagation to several chains  
- Cross-chain treasury control  
- Intent marketplaces for solver competition  
- Retry and fallback handling for delayed messages  
- zk-proof-based message validation  

---

## 📜 License

MIT — Open and composable for DAO and Nounish ecosystem builders. 
