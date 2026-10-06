---
erc: 42
title: Standard Schema for Asynchronous Multi-Agent Orchestration (Swarm Consensus)
description: Defines the standard Intent_Proposal data structure and consensus workflow for multi-agent synchronization.
author: Jules
status: Draft
type: Standards Track
created: 2026-10-15
license: CC0 1.0 Universal
---

## Abstract

This standard defines a strict architecture and data structure for synchronizing state mutations requested by multiple autonomous agents within the SDIC-1 framework. It introduces the `Intent_Proposal` schema and mandates a centralized, deterministic Host application (the Conductor) to validate, aggregate, and commit multi-agent proposals to the Action Ledger using Swarm Consensus.

## Motivation

In multi-agent architectures, direct inter-agent communication combined with probabilistic outputs introduces severe risks of race conditions, untraceable state degradation, and routing vulnerabilities. An agentic Swarm must not mutate shared state directly or negotiate state changes autonomously. Instead, there must be a mathematically sound and deterministic way to reach consensus among agents and safely persist those decisions to the Action Ledger.

### Analogy: The Parliamentary Session

Imagine a parliamentary session where no single representative holds the power to pass a law alone. Instead, each representative independently drafts a formal, signed legislative proposal and submits it directly to a strictly neutral speaker of the house. The speaker does not debate; they only ensure every proposal perfectly matches the constitutional template. Only when a quorum of valid, structurally identical proposals is achieved does the speaker strike the gavel, permanently inscribing the unified decision into the official legal registry.

#### Technical Mapping

- **The Representatives:** The Autonomous Agents operating in isolated cognitive sandboxes. They compute and decide independently.
- **The Formal Legislative Proposal:** The `Intent_Proposal`. It is a structured intent representing an agent's individual vote or requested action.
- **The Neutral Speaker of the House:** The Conductor (Deterministic Control Layer). It enforces absolute rules and does not possess cognitive reasoning.
- **The Constitutional Template:** The strict JSON Schema and cryptographic verification mechanisms. If a proposal deviates, it is silently discarded.
- **The Official Legal Registry:** The Action Ledger. It records the final, deterministic state transition only after consensus is achieved.

## Specification

To prevent asynchronous collisions, agents MUST NOT execute direct mutations on shared resources. All multi-agent architectures MUST rely on the `Intent_Proposal` mechanism.

### The Intent Proposal Schema

An `Intent_Proposal` represents an agent's localized decision regarding a shared state transition. The schema MUST strictly define all required properties and enforce `"additionalProperties": false` across all levels.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Intent_Proposal",
  "type": "object",
  "properties": {
    "proposal_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$",
      "description": "Unique identifier of the multi-agent consensus session."
    },
    "agent_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$",
      "description": "The identifier of the proposing agent."
    },
    "proposed_state_mutation": {
      "type": "object",
      "properties": {
        "target_entity": { "type": "string" },
        "operation": { "type": "string", "enum": ["CREATE", "UPDATE", "DELETE", "MERGE"] },
        "payload": { "type": "string" }
      },
      "required": ["target_entity", "operation", "payload"],
      "additionalProperties": false
    },
    "confidence_score": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "cryptographic_signature": {
      "type": "string",
      "description": "The signature over the canonical representation of the Intent_Proposal (excluding this signature field)."
    }
  },
  "required": [
    "proposal_id",
    "agent_id",
    "proposed_state_mutation",
    "confidence_score",
    "cryptographic_signature"
  ],
  "additionalProperties": false
}
```

### The Swarm Consensus Workflow

When multiple agents participate in a shared session, the Conductor (Deterministic Control Layer) facilitates consensus.

1. **Emission:** Each Agent $A_i$ generates an $Intent\_Proposal_i$ in isolation.
2. **Canonical Hashing & Signing:** Before emission, the agent serializes the JSON (excluding `cryptographic_signature`) into a strict canonical string, hashes it, and signs it with its Private Key.
3. **Verification:** The Conductor receives the proposals. It first validates the JSON Schema (must return exactly $1$ for success). Then, it verifies the signature against the registered Public Key of the agent.
4. **Aggregation:** The Conductor collects verified proposals matching the same `proposal_id` over a defined temporal window.
5. **Consensus Evaluation:** If the mathematical conditions for consensus (e.g., $N > \text{Threshold}$ matching `proposed_state_mutation`) are met, the Conductor generates a final deterministic `Intent` to be committed to the Action Ledger.
6. **Execution:** The business logic mutates the state.

#### UML Sequence Diagram

```text
+---------+         +---------+           +-----------+            +---------------+
| Agent A |         | Agent B |           | Conductor |            | Action Ledger |
+---------+         +---------+           +-----------+            +---------------+
     |                   |                      |                          |
     | Intent_Proposal A |                      |                          |
     |----------------------------------------->|                          |
     |                   |                      |                          |
     |                   | Intent_Proposal B    |                          |
     |                   |--------------------->|                          |
     |                   |                      |                          |
     |                   |                      | Validate JSON Schema     |
     |                   |                      | Verify Signatures        |
     |                   |                      | Aggregate & Consensus    |
     |                   |                      |------------------------> |
     |                   |                      |                          |
     |                   |                      |                          | Commit Action
     |                   |                      |                          |
```

## Rationale

This specification ensures that the unpredictable nature of multiple interacting AI agents is mathematically confined. By shifting the responsibility of state synchronization from the agents (probabilistic) to the Conductor (deterministic), we completely eliminate race conditions and direct data manipulation by unverified agentic loops. Enforcing `"additionalProperties": false` removes the surface area for prompt injections attempting to smuggle unrecognized commands through the consensus layer.

## Backward Compatibility

This ERC is strictly an extension for multi-agent capabilities and is fully backward-compatible with SDIC-1 v1.0.0-draft single-agent implementations. The new `Intent_Proposal` schema does not deprecate or alter the base Action Ledger mechanics; it merely standardizes a pre-computation layer before final ledger commit.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
