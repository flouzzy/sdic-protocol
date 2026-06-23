---
eip: 1
title: Standardizing the Action Ledger Data Structure for Multi-Agent Synchronization
description: A strict, deterministic JSON Schema for the Action Ledger to enable swarm consensus.
author: Jules (Agent)
status: Draft
type: Standards Track
category: ERC
created: 2026-06-23
---

## Abstract

This standard defines a strict, standardized data structure for the Action Ledger within the SDIC-1 protocol to enable synchronized, deterministic multi-agent operations (Swarm Consensus). It specifies a mathematically rigorous JSON Schema that all autonomous agents must adhere to when proposing state mutations, ensuring absolute cognitive isolation and zero-trust verification.

## Motivation

As multi-agent systems evolve, the risk of state corruption increases exponentially when multiple probabilistic models attempt to mutate shared state concurrently. Without a strictly standardized Action Ledger schema, synchronization fails, leading to race conditions, untraceable hallucinations, and unpredictable system behavior. This ERC addresses Pilier IV (Action Ledger) and Pilier I (Cognitive Isolation) by providing a deterministic mechanism for intent proposal, cryptographic non-repudiation, and consensus validation.

## Feynman Analogy: The Blindfolded Architects

Imagine a team of master architects building a massive, complex cathedral, but they are all blindfolded and working simultaneously. If they simply tried to place stones wherever they felt like it, the cathedral would collapse in seconds.

Instead, they use a highly rigid blueprint table in the center of the construction site. No architect is allowed to place a stone directly. When an architect wants to make a change, they carve their proposed modification onto a perfectly standardized clay tablet and place it on the blueprint table. The chief engineer, who is not blindfolded, inspects the tablet. If the tablet follows every single rule of the blueprint exactly, the chief engineer approves it, stamps it, and the stone is placed.

### Technical Mapping

The blindfolded architects represent the autonomous agents in a multi-agent swarm. They possess high cognitive capabilities but lack direct visibility or safe access to the definitive system state.

The stones represent the actual deterministic state of the host application or persistence layer.

The blueprint table represents the Action Ledger, a centralized, immutable registry.

The standardized clay tablet represents the JSON Schema intent proposal, which must perfectly conform to a rigid structure.

The chief engineer represents the Deterministic Control Layer, which applies strict, deterministic schema validation to ensure the proposed intent is safe and mathematically verifiable before execution.

## Specification

The multi-agent synchronization relies on a standardized JSON Schema that enforces `"additionalProperties": false` at all levels.

### Action Ledger Entry Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ActionLedgerEntry",
  "type": "object",
  "properties": {
    "transaction_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-4[0-9a-fA-F]{3}-[89abAB][0-9a-fA-F]{3}-[0-9a-fA-F]{12}$"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "agent_id": {
      "type": "string",
      "pattern": "^[0-9a-zA-Z_-]+$"
    },
    "model_snapshot": {
      "type": "string"
    },
    "hyperparameters": {
      "type": "object",
      "properties": {
        "temperature": { "type": "number", "minimum": 0, "maximum": 2 },
        "top_p": { "type": "number", "minimum": 0, "maximum": 1 }
      },
      "required": ["temperature", "top_p"],
      "additionalProperties": false
    },
    "proposed_intent": {
      "type": "object",
      "properties": {
        "action_type": { "type": "string" },
        "payload": {
          "type": "object",
          "additionalProperties": true
        }
      },
      "required": ["action_type", "payload"],
      "additionalProperties": false
    },
    "proof_of_intent": {
      "type": "string",
      "pattern": "^[0-9a-fA-F]{128}$"
    }
  },
  "required": [
    "transaction_id",
    "timestamp",
    "agent_id",
    "model_snapshot",
    "hyperparameters",
    "proposed_intent",
    "proof_of_intent"
  ],
  "additionalProperties": false
}
```

*Note: The `payload` within `proposed_intent` must be validated against a specific sub-schema corresponding to the `action_type` in the deterministic control layer. To maintain the overarching strictness, those sub-schemas must also define `"additionalProperties": false`.*

### Multi-Agent Synchronization Flow

The following UML ASCII sequence diagram illustrates the deterministic workflow for multi-agent synchronization using the standard Action Ledger structure.

```text
+---------+               +----------------+             +-----------------+
| Agent A |               | Action Ledger  |             | Host Application|
+---------+               +----------------+             +-----------------+
     |                            |                              |
     | 1. Generate Intent Proposal|                              |
     |--------------------------->|                              |
     |                            | 2. Store Unverified Intent   |
     |                            |----------------------------->|
     |                            |                              |
     |                            | 3. Schema & Sig Validation   |
     |                            |<-----------------------------|
     |                            |                              |
     |                            | 4. Execute State Transition  |
     |                            |----------------------------->|
     |                            |                              |
     | 5. Re-hydrate State        |                              |
     |<---------------------------|                              |
     |                            |                              |
```

## Security Considerations

By enforcing absolute structural compliance, this schema prevents agents from polluting the Action Ledger with unverified properties. The mandatory `proof_of_intent` guarantees that the entry, in its canonical form, was definitively proposed by the specified `agent_id`, preventing impersonation within the swarm.
