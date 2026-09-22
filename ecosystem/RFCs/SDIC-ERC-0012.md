---
eip: 12
title: Standardizing the Action Ledger data structure for multi-agent synchronization
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
category: Core
created: 2026-09-22
---

[English](SDIC-ERC-0012.md) | [Français](SDIC-ERC-0012.fr.md)

## Abstract

This standard defines a strict deterministic protocol and data structure for synchronizing multi-agent systems within the SDIC-1 framework. It standardizes the Action Ledger data structure to ensure that autonomous agents can propose state changes safely, relying on cryptographic signatures and strict JSON Schema validation with no unspecified properties permitted.

## Motivation

As enterprise AI systems evolve to incorporate multiple autonomous agents, the risk of state degradation multiplies. Agents operating probabilistically might attempt to mutate shared state concurrently or inject untracked fields. To prevent these vulnerabilities, multi-agent synchronization must not rely on the agents' self-regulation. Instead, it must rely on a deterministic, mathematically verifiable Action Ledger that forces agents to submit standardized, cryptographically signed proposals.

## The Symphony Orchestra Analogy

Imagine a grand symphony orchestra performing a complex piece. Each brilliant musician represents an autonomous agent. If every musician simply plays whenever they wish, the result is cacophony. Furthermore, no musician is allowed to reach out and play another musician's instrument.

To create music, each musician must write their proposed notes on a standardized slip of paper and hand it to a central conductor. The conductor acts as an absolute rule-enforcer, verifying that the slip is formatted perfectly, written legibly, and signed by the submitting musician. If a slip contains any extra, unrequested scribbles, the conductor throws it away immediately. If it is perfect, the conductor transcribes it into the master sheet music. The entire orchestra then reads this master sheet music to adjust their next notes.

In the SDIC-1 architecture:

- **The Musicians:** Autonomous AI agents operating probabilistically.
- **The Instrument Restriction:** Cognitive Isolation (no direct state mutation or network access).
- **The Slip of Paper:** The semantic intent, formatted as a strictly typed JSON object.
- **The Conductor's Verification:** The Deterministic Control Layer applying strict schema validation and cryptographic signature verification.
- **The Thrown-Away Slip:** The rejection of prompt injections or hallucinations by enforcing `"additionalProperties": false`.
- **The Master Sheet Music:** The Action Ledger, acting as the deterministic, append-only synchronization state.

## Specification

The multi-agent synchronization ledger requires a strict JSON Schema for all intent submissions. This schema acts as the standardized slip of paper.

### Intent Schema

Agents MUST format their synchronization proposals exactly according to the following JSON Schema. All objects MUST enforce `"additionalProperties": false` to instantly neutralize prompt injection payloads attempting to append unvalidated data.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentSyncIntent",
  "type": "object",
  "properties": {
    "agent_id": {
      "type": "string",
      "format": "uuid"
    },
    "correlation_id": {
      "type": "string",
      "format": "uuid"
    },
    "proposed_action": {
      "type": "string",
      "enum": ["read_state", "update_context", "delegate_task", "consensus_vote"]
    },
    "payload": {
      "type": "object",
      "properties": {
        "target_agent_id": { "type": "string", "format": "uuid" },
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

### Cryptographic Identity & Proof of Intent

To guarantee non-repudiation and prevent relabeling attacks, the `signature` field MUST be derived from the exact canonical representation of the intent object, strictly excluding the `signature` field itself.

1. **Canonicalization:** The JSON object is stripped of the `signature` key. The remaining structure is serialized using predictable key ordering (lexicographical sorting) and removal of insignificant whitespace (JSON Canonicalization Scheme).
2. **Hashing:** Compute the SHA-256 hash of the canonicalized UTF-8 string.
3. **Signing:** The agent signs the hash using its designated private key.
4. **Verification:** The Deterministic Control Layer verifies the signature against the agent's known public key.

### Sequence Diagram

The interaction between an agent and the deterministic Action Ledger is strictly asynchronous and validated by the host application.

```text
+----------+                     +-----------------------+                  +---------------+
| Agent A  |                     | Deterministic Control |                  | Action Ledger |
+----------+                     +-----------------------+                  +---------------+
     |                                     |                                    |
     | 1. Generate Canonical Intent        |                                    |
     |------------------------------------>|                                    |
     |                                     | 2. Verify Schema Compliance        |
     |                                     | (Drop if invalid)                  |
     |                                     |                                    |
     |                                     | 3. Cryptographic Validation        |
     |                                     | (Drop if signature invalid)        |
     |                                     |                                    |
     |                                     | 4. Append to Ledger                |
     |                                     |----------------------------------->|
     |                                     |                                    |
     |                                     | 5. Broadcast State Update          |
     |<-------------------------------------------------------------------------|
     |                                     |                                    |
```

## Security Considerations

The primary attack vector in multi-agent orchestration is malicious context pollution via prompt injection, where a compromised agent attempts to propagate hostile instructions to the swarm.

This risk is heavily mitigated by:

1. Strict schema validation (`"additionalProperties": false`) filtering unstructured data and neutralizing hidden payloads.
2. Cryptographic non-repudiation binding every action to a specific agent's key, securing against man-in-the-middle relabeling.
3. Deterministic state transition engines acting as the only entities capable of modifying shared context, isolating the probabilistic models.

## Backward Compatibility

This RFC extends the initial SDIC-1 draft without breaking existing implementations. Single-agent setups can safely ignore `correlation_id` and swarm coordination actions, maintaining compatibility with the core Pilier 4: Action Ledger concept described in the root specification.

## Copyright

Copyright and related rights waived via CC0.
