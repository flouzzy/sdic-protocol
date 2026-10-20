---
title: "SDIC-ERC-0020: Standardized Action Ledger Data Structure for Multi-Agent Synchronization"
author: "Jules <jules@example.com>"
status: Draft
type: Standards Track
created: 2026-10-20
---

[English](SDIC-ERC-0020.md) | [Français](SDIC-ERC-0020.fr.md)

## Abstract

This standard proposes an architectural pattern and a strict JSON schema for the "Action Ledger" component of the SDIC-1 framework to facilitate secure, deterministic multi-agent synchronization (Swarm Consensus). It mathematically confines inter-agent communication, ensuring that multiple probabilistic LLMs can orchestrate asynchronous tasks without directly mutating shared state or introducing race conditions.

## 1. Introduction

Imagine an air traffic control tower coordinating a fleet of autonomous airplanes. If the airplanes attempt to communicate and coordinate their flight paths directly with each other over an open radio frequency, the result is chaos. A transmission error, a misunderstood signal, or a malicious interference could cause two planes to attempt occupying the same airspace simultaneously.

To prevent this, we introduce the centralized flight manifest and the dispatcher. The airplanes (the autonomous agents) are strictly forbidden from directing each other. Instead, when an airplane wants to change its altitude, it sends a highly structured, cryptographically signed request (the Intent) to the dispatcher (the Deterministic Control Layer). The dispatcher verifies the request against strict aviation rules. If valid, the dispatcher updates the central flight manifest (the Action Ledger). All other airplanes continuously read from this immutable manifest to understand the current state of the airspace and adjust their behavior accordingly.

In the SDIC-1 architecture:

- **The autonomous airplanes** represent the probabilistic AI agents. They are intelligent but cannot directly alter the shared system state.
- **The dispatcher** represents the Deterministic Control Layer, applying strict JSON schema validation.
- **The central flight manifest** is the Standardized Action Ledger. It serves as the single, immutable source of truth that synchronizes the swarm deterministically.

## 2. Rationale & Architectural Impact

As multi-LLM architectures evolve, agents must collaborate seamlessly while adhering to the cognitive isolation mandated by SDIC-1 (Pilier I). Current implementations lack a standardized medium for agents to broadcast and validate intents across a Swarm Consensus safely. Unstructured agent-to-agent communication poses significant risks:

- **State Drift:** Without a centralized truth, agents can desynchronize, acting on outdated or conflicting context.
- **Prompt Injection Propagation:** Direct communication allows a hijacked agent to spread a malicious prompt payload to the rest of the swarm.
- **Race Conditions:** Simultaneous actions without a deterministic locking mechanism lead to unpredictable state mutations.

A standardized Action Ledger schema ensures immutable, deterministic, and cryptographically verified agent coordination. By forcing `"additionalProperties": false`, the schema instantly dissipates malformed intents and neutralize prompt injection payloads attempting to append unvalidated data.

## 3. Proposed Specification Changes

### 3.1. Target Component

This specification targets **Pilier IV: The Action Ledger**, defining a deterministic structure for multi-agent intent proposals and consensus.

### 3.2. Multi-Agent Action Ledger JSON Schema

Agents MUST format their synchronization proposals exactly according to the following JSON Schema. All objects MUST enforce `"additionalProperties": false`. Cryptographic signatures MUST validate a canonical representation of the entire entry (excluding the signature field itself) to prevent replay and relabeling attacks.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentActionLedgerEntry",
  "type": "object",
  "properties": {
    "transaction_id": {
      "type": "string",
      "format": "uuid"
    },
    "correlation_id": {
      "type": "string",
      "format": "uuid",
      "description": "Used to link multiple intents in a distributed saga"
    },
    "agent_id": {
      "type": "string",
      "format": "uuid"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "proposed_action": {
      "type": "string",
      "enum": ["propose_state_mutation", "acknowledge_state", "request_consensus"]
    },
    "payload": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "target_resource": { "type": "string" },
        "mutation_type": { "type": "string", "enum": ["create", "update", "delete"] },
        "mutation_data": { "type": "string", "description": "Deterministic string representation of the change" }
      },
      "required": ["target_resource", "mutation_type", "mutation_data"]
    },
    "proof_of_intent": {
      "type": "string",
      "description": "Cryptographic signature of the canonical intent"
    }
  },
  "required": [
    "transaction_id",
    "correlation_id",
    "agent_id",
    "timestamp",
    "proposed_action",
    "payload",
    "proof_of_intent"
  ],
  "additionalProperties": false
}
```

### 3.3. Swarm Consensus Sequence Diagram

The following UML ASCII sequence diagram illustrates the deterministic propagation of state changes through the Action Ledger, preventing direct agent-to-agent mutation.

```text
+---------+               +-----------------------+              +-----------------+               +---------+
| Agent A |               | Deterministic Control |              | Action Ledger   |               | Agent B |
+---------+               +-----------------------+              +-----------------+               +---------+
     |                                |                                   |                             |
     | 1. Emits Intent (JSON)         |                                   |                             |
     |------------------------------->|                                   |                             |
     |                                | 2. Schema Validation              |                             |
     |                                |    (additionalProperties: false)  |                             |
     |                                |-----------------------------      |                             |
     |                                |                            |      |                             |
     |                                |<----------------------------      |                             |
     |                                |                                   |                             |
     |                                | 3. Cryptographic Verification     |                             |
     |                                |    (Canonical Payload)            |                             |
     |                                |-----------------------------      |                             |
     |                                |                            |      |                             |
     |                                |<----------------------------      |                             |
     |                                |                                   |                             |
     |                                | 4. Commit to Ledger               |                             |
     |                                |---------------------------------->|                             |
     |                                |                                   | 5. State Re-hydration       |
     |                                |                                   |---------------------------->|
     |                                |                                   |                             |
```

## 4. Backward Compatibility

This specification is fully backward-compatible. Systems implementing the `v1.0.0-draft` of SDIC-1 can support multi-agent consensus by introducing the `correlation_id` and standardizing the ledger payload structure without requiring changes to the underlying AI Sandbox or existing deterministic filtering rules. Single-agent systems can safely ignore the multi-agent specific fields.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
