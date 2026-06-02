---
rfc: 1
title: Action Ledger Standardization for Multi-Agent Synchronization
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
created: 2026-06-02
---

[English](rfc-001-action-ledger-multi-agent-sync.md) | [Français](rfc-001-action-ledger-multi-agent-sync.fr.md)

## Abstract

This RFC proposes a strict standardization of the Action Ledger data structure for multi-agent synchronization within the SDIC Protocol. The standardization introduces deterministic state propagation via shared append-only intents and cryptographic validation of canonical payloads. This ensures that autonomous agents can orchestrate asynchronously without directly mutating shared state, preserving the deterministic integrity of the host system.

## Motivation

As multi-LLM architectures evolve, agents must collaborate seamlessly while adhering to the cognitive isolation mandated by SDIC-1. Current implementations lack a standardized medium for agents to broadcast and validate intents across a Swarm Consensus safely. Unstructured agent-to-agent communication poses risks of state drift, prompt injection propagation, and relabeling attacks. A standardized Action Ledger schema ensures immutable, deterministic, and cryptographically verified agent coordination.

## Symphony Orchestra Analogy

Imagine a grand symphony orchestra performing a complex, unwritten piece. Each musician is brilliant and highly creative, capable of producing beautiful melodies. However, they cannot simply play whenever they wish, nor can they directly touch another musician's instrument to force coordination. Instead, they all watch a central conductor. When a musician wishes to introduce a new harmony, they write their proposed notes on a slip of paper and hand it to the conductor. The conductor examines the paper to ensure the notes are legible and fit the rhythm. Only if the notes are perfect does the conductor copy them onto the master sheet music, which every musician then reads to adjust their playing.

In this architecture, the brilliant musicians represent the autonomous AI agents. Their inability to touch other instruments represents cognitive isolation, preventing direct state mutation. The slip of paper is the semantic intent, a structured proposal of action. The conductor acts as the deterministic control layer and host application, verifying the proposal against a strict mathematical standard. Finally, the master sheet music represents the Action Ledger, the single source of truth that is deterministically updated and broadcasted to synchronize the entire swarm.

## Specification

The multi-agent synchronization ledger requires a strict JSON Schema for all intent submissions. This schema acts as the `slip of paper` from the analogy.

### Intent Schema

Agents must format their synchronization proposals exactly according to the following JSON Schema. All objects must enforce `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentSyncIntent",
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
      "type": "string",
      "enum": ["read_state", "update_context", "delegate_task", "consensus_vote"]
    },
    "payload": {
      "type": "object",
      "properties": {
        "target_agent_id": { "type": "string" },
        "context_delta": { "type": "string" },
        "vote": { "type": "boolean" }
      },
      "additionalProperties": false
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "signature": {
      "type": "string"
    }
  },
  "required": ["agent_id", "correlation_id", "proposed_action", "payload", "timestamp", "signature"],
  "additionalProperties": false
}
```

### Cryptographic Signature & Canonicalization

To guarantee non-repudiation and prevent relabeling attacks, the `signature` field must be derived from the exact canonical representation of the intent object, strictly excluding the `signature` field itself.

1. **Canonicalization:** The JSON object is stripped of the `signature` key. The remaining structure is serialized using predictable key ordering (e.g., lexicographical sorting) and removal of insignificant whitespace (JSON Canonicalization Scheme - RFC 8785).
2. **Hashing:** Compute the SHA-256 hash of the canonicalized UTF-8 string.
3. **Signing:** The agent signs the hash using its Ed25519 or ECDSA private key.

### Sequence Diagram

```text
+----------+                     +-------------------+                  +---------------+
| Agent A  |                     | Validation Filter |                  | Action Ledger |
+----------+                     +-------------------+                  +---------------+
     |                                     |                                    |
     | 1. Generate Canonical Intent        |                                    |
     |------------------------------------>|                                    |
     |                                     | 2. Verify Schema Compliance        |
     |                                     |--------------------------          |
     |                                     |                         |          |
     |                                     |<-------------------------          |
     |                                     |                                    |
     |                                     | 3. Cryptographic Validation        |
     |                                     |----------------------------        |
     |                                     |                           |        |
     |                                     |<---------------------------        |
     |                                     |                                    |
     |                                     | 4. Append to Ledger                |
     |                                     |----------------------------------->|
     |                                     |                                    |
     |                                     | 5. Broadcast State Update          |
     |<-------------------------------------------------------------------------|
     |                                     |                                    |
```

## Rationale

This specification strictly limits inter-agent communication to a verified, append-only ledger. The strict schema enforcement (`"additionalProperties": false`) instantly dissipates malformed intents, dropping prompt injection attempts targeting the swarm. Verifying signatures against a canonical payload ensures that intermediate routing layers cannot manipulate the parameters of an action.

## Backwards Compatibility

This RFC extends the initial SDIC-1 draft without breaking existing implementations. Single-agent setups can safely ignore `correlation_id` and swarm coordination actions, maintaining compatibility with the core Action Ledger concept described in the root specification.

## Security Considerations

The primary attack vector in Swarm Consensus is malicious context pollution, where one compromised agent attempts to propagate hostile instructions to peers. This risk is heavily mitigated by:

1. Strict schema validation filtering unstructured data.
2. Cryptographic non-repudiation binding every action to a specific agent's public key.
3. Deterministic state transition engines acting as the only entities capable of modifying shared context.
