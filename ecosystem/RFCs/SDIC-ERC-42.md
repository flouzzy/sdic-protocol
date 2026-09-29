---
eip: 42
title: Standard Schema for Asynchronous Multi-Agent Orchestration
description: A standard data structure and consensus workflow for multi-agent asynchronous orchestration within the SDIC-1 protocol.
author: Jules
status: Draft
type: Standards Track
category: ERC
created: 2026-09-01
---

## Abstract

This standard defines the `Intent_Proposal` schema and consensus workflow required to synchronize multiple autonomous agents operating within the SDIC-1 framework. It addresses the architectural gap of multi-agent state management by standardizing the Action Ledger data structure to prevent unpredictable state mutations during asynchronous agent orchestration.

## Motivation

As AI-native development shifts towards multi-agent (Swarm) architectures, a single agent's probabilistic outputs are compounded by asynchronous interactions with other agents. Direct inter-agent communication and state modification lead to race conditions, untraceable state degradation, and routing vulnerabilities. This ERC formalizes the synchronization of multi-agent intents through a deterministic, cryptographically secure consensus layer.

## Specification

### 1. The Symphony Conductor Analogy

Imagine a massive symphony orchestra where each musician is highly skilled but completely deaf to the others. If they all play simultaneously without coordination, the result is chaotic noise.

To create music, we introduce a silent Conductor standing at a central podium. The musicians never speak to each other directly. Instead, each musician writes down the exact note they intend to play on a piece of paper, signs their name on it, and hands it to the Conductor. The Conductor reads all the proposed notes, checks them against the master sheet music, and only if the notes harmonize, does the Conductor strike the baton, allowing the entire orchestra to play those notes simultaneously.

### 2. Technical Mapping

- **The Musicians (Autonomous Agents):** These are the individual AI agents. They operate in isolation and cannot communicate directly with one another or modify the shared state.
- **The Written Note (Intent Proposal):** This is the `Intent_Proposal` JSON object. It represents the intended action an agent wishes to perform.
- **The Signature (Cryptographic Identity):** Each agent cryptographically signs their canonical `Intent_Proposal` to ensure non-repudiation and prevent tampering.
- **The Conductor (Deterministic Control Layer):** The deterministic host application that collects, validates, and evaluates the intents against the system's consensus rules (the master sheet music).
- **The Strike of the Baton (Action Ledger Commit):** The final, atomic state transition executed by the host only after consensus is achieved.

### 3. Intent Proposal Schema

The `Intent_Proposal` must strictly adhere to the following JSON Schema. To guarantee determinism and mitigate injection risks, all objects enforce `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Intent_Proposal",
  "type": "object",
  "properties": {
    "agent_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$"
    },
    "correlation_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$"
    },
    "proposed_action": {
      "type": "object",
      "properties": {
        "operation": {
          "type": "string"
        },
        "parameters": {
          "type": "object",
          "additionalProperties": false
        }
      },
      "required": ["operation", "parameters"],
      "additionalProperties": false
    },
    "cryptographic_signature": {
      "type": "string"
    }
  },
  "required": ["agent_id", "correlation_id", "proposed_action", "cryptographic_signature"],
  "additionalProperties": false
}
```

### 4. Consensus Workflow Diagram

The following UML ASCII sequence diagram illustrates the lifecycle of multi-agent intent proposals through the deterministic consensus layer.

```text
+---------+       +---------+       +-------------------+       +---------------+
| Agent A |       | Agent B |       | Host (Conductor)  |       | Action Ledger |
+---------+       +---------+       +-------------------+       +---------------+
     |                 |                      |                         |
     |--- Intent A --->|                      |                         |
     |                 |--- Intent B -------->|                         |
     |                 |                      |                         |
     |                 |                      |-- Validate Schema ----->|
     |                 |                      |                         |
     |                 |                      |-- Validate Sigs ------->|
     |                 |                      |                         |
     |                 |                      |-- Evaluate Consensus -->|
     |                 |                      |                         |
     |                 |                      |-- Commit State (Atomic)>|
     |                 |                      |                         |
+---------+       +---------+       +-------------------+       +---------------+
```

## Security Considerations

- The `cryptographic_signature` must be generated over the canonical representation of the `Intent_Proposal` (excluding the signature field itself) to prevent relabeling.
- The Host must enforce consensus timeout mechanisms to prevent indefinite hanging if an agent fails to submit an intent.
