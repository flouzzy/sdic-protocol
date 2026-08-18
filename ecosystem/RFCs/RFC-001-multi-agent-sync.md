# ERC-1001: Multi-Agent Action Ledger Synchronization Standard

## Preamble

**ERC:** 1001
**Title:** Multi-Agent Action Ledger Synchronization Standard
**Author:** Charles EDOU NZE
**Type:** Standards Track
**Category:** SDIC-1 Extension
**Status:** Draft
**Created:** 2026-08-18

## Abstract

This RFC proposes a standardized data structure and synchronization mechanism for the Action Ledger within the SDIC-1 architecture to support Swarm Consensus and multi-agent orchestration. It defines a strict, deterministic schema for agent intent proposals and consensus verification to prevent unauthorized state mutations in shared environments.

## Motivation

As AI-native architectures evolve from single-agent integrations to multi-agent swarms, the risk of state corruption multiplies. A single deterministic host may need to reconcile conflicting intents from multiple probabilistic agents. Without a standardized, cryptographically verifiable ledger entry format specifically designed for multi-agent synchronization, agents might inadvertently overwrite each other's state or bypass consensus mechanisms. This standard enforces structural integrity and cryptographic non-repudiation for every step of a multi-agent workflow.

## The Symphony Orchestra Analogy

Imagine a world-class symphony orchestra where every musician is incredibly talented but entirely deaf. They can perfectly play their own instrument but cannot hear what anyone else is playing. If they all started playing at once based only on their individual sheet music, the result would be chaotic noise.

To create beautiful music, they rely on a central conductor and a massive, highly visible metronome and shared scoreboard. A musician doesn't just play a note; they write their intended note on a card, sign it, and hand it to the conductor. The conductor, who can see everyone's cards, verifies the signatures, checks if the notes fit the current measure of the symphony, and then officially pins the approved notes to the shared scoreboard for the next beat. Only the notes on the scoreboard are actually "played" in reality.

**Technical Mapping:**

- **The Deaf Musicians:** The autonomous, probabilistic AI agents operating in isolation (Sandbox).
- **The Intended Note on a Signed Card:** The `MultiAgentActionLedgerEntry` (the JSON intent object), cryptographically signed by the agent's private key.
- **The Conductor:** The Deterministic Control Layer of the Host Application, responsible for strict validation (Semantic Determinism).
- **Checking if notes fit the measure:** Swarm Consensus validation function.
- **The Shared Scoreboard:** The immutable Action Ledger.
- **Playing the Note:** The State Transition Engine executing the validated intent.

## Specification

### 1. The Multi-Agent Action Ledger Entry Schema

Any multi-agent proposal submitted to the host application MUST adhere to the following strict JSON schema. The host MUST reject any payload that fails validation.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentActionLedgerEntry",
  "type": "object",
  "properties": {
    "proposal_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$",
      "description": "Unique UUIDv4 for the proposal."
    },
    "agent_id": {
      "type": "string",
      "description": "Public identifier of the proposing agent."
    },
    "swarm_id": {
      "type": "string",
      "description": "Identifier of the consensus group."
    },
    "action_type": {
      "type": "string",
      "enum": ["propose_state_mutation", "vote_commit", "vote_reject"]
    },
    "payload": {
      "type": "object",
      "description": "The deterministic state mutation requested.",
      "additionalProperties": false
    },
    "nonce": {
      "type": "integer",
      "description": "Strictly increasing integer to prevent replay attacks."
    },
    "signature": {
      "type": "string",
      "description": "Cryptographic signature of the canonical proposal."
    }
  },
  "required": [
    "proposal_id",
    "agent_id",
    "swarm_id",
    "action_type",
    "payload",
    "nonce",
    "signature"
  ],
  "additionalProperties": false
}
```

### 2. Canonical Representation and Cryptographic Signature

To prevent relabeling or replay attacks by malicious actors altering the payload in transit, the `signature` field MUST validate a canonical representation of the object.

1. **Canonicalization:** The host and the agent MUST generate a strict deterministic JSON string of the entry, completely omitting the `signature` key. Keys must be sorted alphabetically, and whitespace must be stripped.
2. **Hashing:** The canonical string is hashed using SHA-256.
3. **Signing:** The hash is signed using the agent's Private Key (e.g., Ed25519 or ECDSA).

Let $I$ be the Intent object.
$Canonical(I)$ = JSON string of $I$ excluding the `signature` field, sorted keys, no whitespace.
$\sigma = Sign_{AgentPrivKey}(Hash_{SHA256}(Canonical(I)))$

The host MUST verify this signature against the registry of approved agents for the given `swarm_id` before admitting the proposal to the Swarm Consensus function.

## Rationale

By enforcing `additionalProperties: false` globally and requiring cryptographic signatures over a canonicalized payload excluding the signature itself, this RFC completely eliminates the surface area for unspecified key injections and transit-based payload tampering in multi-agent environments.

## Backwards Compatibility

This is a new extension to the SDIC-1 standard (v1.0.0-draft) explicitly targeting multi-agent swarm configurations. It does not break existing single-agent implementations of the Action Ledger, as it introduces a new specific schema for swarm proposals rather than modifying the base temporal auditability table.
