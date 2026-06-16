---
title: "SDIC-RFC-0004: Standardized Action Ledger for Asynchronous Multi-Agent Orchestration"
author: "Jules <jules@example.com>"
status: Draft
type: Standards Track
created: 2026-06-01
---

## Abstract

This RFC standardizes the "Action Ledger" data structure for asynchronous multi-agent orchestration within the SDIC-1 framework. To prevent state corruption when multiple autonomous agents operate concurrently, this specification introduces a deterministic, cryptographically secure schema for logging intents and achieving swarm consensus before state transition.

## 1. Introduction

As multi-agent systems evolve, the probability of state collision and unpredictable cascading effects increases exponentially. When multiple probabilistic agents act upon a shared state, traditional monolithic databases fail to provide the cognitive isolation required by SDIC-1.

Imagine an Air Traffic Control (ATC) system managing a swarm of autonomous delivery drones. The drones themselves (the probabilistic agents) never decide to land simultaneously on the same runway. Instead, each drone calculates its optimal trajectory and sends a formal Landing Request (the intent) to the control tower. The control tower (the deterministic host) acts as an immutable log and sequencer. It verifies every request, ensures that no two drones attempt to use the runway simultaneously based on rigid physical rules, and then clears the landing one by one. The drones do not coordinate directly with each other by altering their flight paths arbitrarily; they synchronize through the central, deterministic ledger of the control tower.

Mapping this analogy to our technical architecture:

- **The Drones:** These are the autonomous LLM agents operating within their isolated sandboxes. They calculate optimal solutions based on probabilistic reasoning but lack direct authority to execute state changes.
- **The Landing Request:** This is the strictly defined `Intent_Proposal` conforming to a JSON Schema. It represents the AI's semantic intent to modify the state.
- **The Control Tower:** This represents the Host Application. It receives requests, validates them against deterministic rules, and sequences them.
- **The Immutable Log:** This is the Action Ledger. Every validated `Intent_Proposal` is recorded here cryptographically before any actual state change occurs.
- **Clearance to Land:** This is the Swarm Consensus and subsequent State Transition. The host guarantees that only non-conflicting, mathematically verified intents are executed.

## 2. Rationale & Architectural Impact

The primary vulnerability in multi-LLM workflows is the lack of synchronized, deterministic arbitration between conflicting agent intents. If two agents attempt to mutate a resource concurrently without a strict structure, the system enters a non-deterministic state.

By enforcing a standardized, cryptographic Action Ledger, we achieve:

1. **Temporal Determinism:** All agent intents are strictly serialized.
2. **Cryptographic Non-Repudiation:** Signatures ensure that intents cannot be altered or spoofed after emission.
3. **Swarm Consensus Foundation:** Provides the exact data structure required for agents to read past intents and formulate coherent subsequent proposals without direct agent-to-agent chatter.

## 3. Specification

### 3.1. Standardized Action Ledger Entry Schema

The following JSON Schema defines the required structure for any entry committed to the Action Ledger in a multi-agent context. Strict adherence is enforced (`"additionalProperties": false`).

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ActionLedgerEntry",
  "type": "object",
  "properties": {
    "transaction_id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique, immutable identifier of the transaction."
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "System-level precise temporal marker."
    },
    "agent_id": {
      "type": "string",
      "description": "Identifier of the autonomous agent emitting the intent."
    },
    "model_snapshot": {
      "type": "string",
      "description": "Exact version of the model used."
    },
    "intent_type": {
      "type": "string",
      "enum": ["READ", "MUTATE", "PROPOSE", "VOTE"]
    },
    "payload": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "target_resource": { "type": "string" },
        "action": { "type": "string" },
        "parameters": { "type": "object", "additionalProperties": true }
      },
      "required": ["target_resource", "action"]
    },
    "proof_of_intent": {
      "type": "string",
      "description": "Cryptographic signature of the canonical intent."
    }
  },
  "required": [
    "transaction_id",
    "timestamp",
    "agent_id",
    "model_snapshot",
    "intent_type",
    "payload",
    "proof_of_intent"
  ],
  "additionalProperties": false
}
```

### 3.2. Asynchronous Multi-Agent Orchestration Flow

The interaction between agents and the host must follow a strict asynchronous pattern, mediated entirely by the Action Ledger.

```text
+---------+          +-------------------+          +-------------+
| Agent A |          | Action Ledger (AL)|          | Host System |
+---------+          +-------------------+          +-------------+
     |                         |                           |
     | 1. Read Current State   |                           |
     |------------------------>|                           |
     |                         |                           |
     | 2. Compute Intent       |                           |
     | (Probabilistic)         |                           |
     |                         |                           |
     | 3. Submit Intent        |                           |
     | (Signed JSON)           |                           |
     |------------------------>|                           |
     |                         | 4. Validate Schema & Sig. |
     |                         |-------------------------->|
     |                         |                           |
     |                         |                           | 5. If Valid, Sequence
     |                         |                           |    and Attempt Execution
     |                         |                           |
     |                         | 6. Record Outcome         |
     |                         |<--------------------------|
     |                         |                           |
     | 7. Read Updated State   |                           |
     |<------------------------|                           |
     |                         |                           |
```

## 4. Security Considerations

- **Canonicalization:** The `proof_of_intent` signature MUST validate a canonicalized version of the JSON object (e.g., sorted keys, no whitespace) minus the `proof_of_intent` field itself.
- **Replay Attacks:** The `transaction_id` and `timestamp` fields must be strictly validated by the host to ensure an intent is not executed more than once.

## 5. Backward Compatibility

This RFC extends the initial concept of the Action Ledger outlined in SDIC-1 v1.0.0-draft. It introduces more rigid structures for multi-agent capabilities but is fully backward compatible with single-agent systems that implement the core SDIC-1 philosophy.
