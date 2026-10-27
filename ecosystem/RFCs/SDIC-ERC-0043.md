[Français](SDIC-ERC-0043.fr.md) | [English](SDIC-ERC-0043.md)

# SDIC-ERC-0043: Action Ledger Data Structure for Multi-Agent Synchronization

- **Author:** Jules (AI Systems Engineer)
- **Status:** Draft
- **Type:** Standards Track
- **Category:** Architecture (Pilier IV)
- **Created:** 2026-10-27

---

## Abstract

This standard defines a strict schema and protocol for the "Action Ledger" data structure to facilitate asynchronous multi-agent orchestration and synchronization. By establishing a canonical, deterministic JSON Schema for multi-agent intents and leveraging cryptographic signatures over these structures, this ERC ensures that agents can safely coordinate state transitions without direct access to each other or the underlying system state.

## 1. Motivation

As critical enterprise systems increasingly adopt multi-agent (Swarm) architectures, the lack of a standardized data structure for cross-agent communication and state synchronization introduces significant vulnerabilities. Without a strictly typed ledger for recording agent intents, the risk of semantic drift, hallucinated state variables, and prompt-injection-driven execution increases.

The goal of SDIC-ERC-0043 is to standardize the Pilier IV Action Ledger payload specifically for Swarm Consensus, ensuring that any proposal to mutate shared state must pass through a strict, zero-trust schema validation layer before being recorded and acted upon.

## 2. Analogy

Imagine a high-security corporate boardroom where multiple executives (agents) must agree on a major financial decision. In a flawed system, they might just yell their ideas across the room, leading to misunderstandings, manipulation, or unauthorized actions by rogue participants.

Instead, in this strict protocol, no executive is allowed to speak directly to another or to the company's bank. When an executive wants to propose an action, they must write their proposal on a standardized, tamper-proof carbon-copy form. This form requires specific fields (Action, Target, Reason). The executive must then physically stamp the form with their unique, verifiable wax seal (Cryptographic Signature). This form is then submitted to an impartial, automated clerk (Deterministic Control Layer). The clerk only checks if the form is filled out perfectly and the seal is valid. If it is, the proposal is filed in the transparent public ledger (Action Ledger), where other executives can read it and submit their own forms in response.

### 2.1. Analogy Mapping

- **The Executives (Agents):** The autonomous probabilistic AI models participating in the swarm.
- **Yelling Across the Room (Direct Communication):** Unsafe, unstructured LLM-to-LLM prompt interactions.
- **The Standardized Carbon-Copy Form (Action Ledger Schema):** The strict JSON Schema defined in this RFC, ensuring all proposals adhere to an exact structural format.
- **The Wax Seal (Cryptographic Signature):** The deterministic proof of intent ($\sigma$), validating the canonical form of the proposal.
- **The Impartial Clerk (Deterministic Control Layer):** The host application's validation logic, which rejects non-conforming or incorrectly signed proposals.
- **The Public Ledger (Action Ledger):** The append-only, temporal state history that agents read from to synchronize their context.

## 3. Specification

### 3.1. Standardized Action Ledger Schema

All multi-agent synchronization proposals MUST conform strictly to the following JSON Schema. To prevent undefined keys, `"additionalProperties": false` MUST be enforced at all object levels.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentSyncProposal",
  "type": "object",
  "properties": {
    "proposal_id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique identifier for the proposal."
    },
    "agent_id": {
      "type": "string",
      "description": "The unique identifier of the proposing agent."
    },
    "action_type": {
      "type": "string",
      "enum": ["propose_state_change", "endorse_proposal", "reject_proposal"],
      "description": "The nature of the agent's intent."
    },
    "target_state_key": {
      "type": "string",
      "description": "The specific state variable or resource being targeted."
    },
    "proposed_value": {
      "type": ["string", "number", "boolean", "null"],
      "description": "The desired new value for the state key (if applicable)."
    },
    "justification": {
      "type": "string",
      "description": "Semantic reasoning behind the proposal."
    },
    "reference_proposal_id": {
      "type": ["string", "null"],
      "format": "uuid",
      "description": "ID of a prior proposal this action relates to (e.g., for endorsements)."
    },
    "signature": {
      "type": "string",
      "description": "Cryptographic signature over the canonical payload."
    }
  },
  "required": [
    "proposal_id",
    "agent_id",
    "action_type",
    "target_state_key",
    "justification",
    "signature"
  ],
  "additionalProperties": false
}
```

### 3.2. Cryptographic Canonicalization

To prevent replay and relabeling attacks within the swarm, the signature must validate a canonical representation of the proposal.

1. Create a `CanonicalPayload` object by removing the `signature` field from the proposal.
2. Serialize `CanonicalPayload` deterministically (e.g., sort keys lexicographically, remove whitespace).
3. Compute the signature: $\sigma = Sign_{AgentPrivKey}(Hash(CanonicalPayload))$
4. Append $\sigma$ to the final proposal object as the `signature` field.

### 3.3. Orchestration Sequence

The sequence of operations for state synchronization must strictly follow this deterministic path.

```text
+----------+             +-----------------+             +---------------+
| Agent A  |             | Control Layer   |             | Action Ledger |
+----+-----+             +--------+--------+             +-------+-------+
     |                            |                              |
     | 1. Generate Proposal JSON  |                              |
     | 2. Canonicalize & Sign     |                              |
     | 3. Submit Payload          |                              |
     +--------------------------->|                              |
     |                            | 4. Validate Schema (Reject   |
     |                            |    if extra keys exist)      |
     |                            | 5. Validate Signature        |
     |                            | 6. Append to Ledger          |
     |                            +----------------------------->|
     |                            |                              |
     |                            |                              |
+----+-----+                      |                              |
| Agent B  |                      |                              |
+----+-----+                      |                              |
     | 7. Re-hydrate Context      |                              |
     |<---------------------------+------------------------------+
     | 8. Analyze New Proposal    |                              |
     | 9. Generate Endorsement    |                              |
     +--------------------------->|                              |
     |                            | 10. Validate Endorsement     |
     |                            | 11. Append to Ledger         |
     |                            +----------------------------->|
```

## 4. Backward Compatibility

This standard is fully backward compatible with the core Pilier IV definitions outlined in v1.0.0-draft of the SDIC-1 specification. It extends the concept of the Action Ledger by providing a specialized payload schema for Swarm environments without modifying the underlying ledger requirements (such as transaction IDs or temporal markers).

## 5. License

This document and its contents are released under the CC0 1.0 Universal (Public Domain Dedication).
