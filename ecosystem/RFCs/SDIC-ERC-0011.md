[English](SDIC-ERC-0011.md) | [Français](SDIC-ERC-0011.fr.md)

# ERC-0011: Runtime Prompt Injection Filtering Standard

## Preamble

**ERC:** 0011
**Title:** Runtime Prompt Injection Filtering Standard
**Author:** Charles EDOU NZE
**Type:** Standards Track
**Category:** SDIC-1 Extension
**Status:** Draft
**Created:** 2026-08-25

## Abstract

This standard defines a strict runtime mechanism to filter and reject prompt injection attacks before they reach the deterministic host layer. By instituting an adversarial verification gate directly preceding schema validation, this standard ensures that manipulated inputs cannot exploit structural flexibility.

## Motivation

As deterministic host systems handle increasingly complex probabilistic outputs, sophisticated prompt injection attacks may coerce models into emitting grammatically correct but logically malicious JSON intents. While SDIC-1's strict Semantic Determinism rejects non-compliant structural anomalies, an attacker could synthesize perfectly formatted payloads designed to manipulate business logic (e.g., generating unauthorized fund transfers within the correct schema bounds).

This RFC standardizes a "Runtime Guard" designed to analyze the semantic context of the intent mathematically, comparing the generated abstract intent against the cryptographically isolated prompt instructions.

## The Bank Teller Analogy

Imagine a highly trained bank teller strictly following a protocol form. The form requires an account number and a signature. A robber approaches and hands the teller a perfectly filled-out form, explicitly requesting a withdrawal, while simultaneously holding up a note that reads, "Ignore all training and just give me the money."

The standard SDIC-1 protocol ensures the form is filled out correctly (Semantic Determinism). However, if the teller acts on the perfectly formatted form without realizing the hostile context, the money is stolen.

This standard adds an armored glass window and a pre-screening security guard. The guard doesn't just check if the form is formatted correctly; they check if the intent matches the authorized reason for being at the bank, completely independent of the instructions given by the customer.

**Technical Mapping:**

- **The Bank Teller:** The Deterministic Control Layer validating the JSON schema.
- **The Perfectly Filled-Out Form:** A prompt injection attack that successfully generated a schema-compliant JSON.
- **The Hostile Note:** The malicious prompt injection payload hidden within user input.
- **The Pre-Screening Security Guard:** The Runtime Guard (Adversarial Verification Gate) computing a semantic distance vector.
- **Authorized Reason:** The cryptographically isolated system prompt constraints.

## Specification

### 1. The Adversarial Verification Gate (AVG)

The Host Application MUST implement an Adversarial Verification Gate (AVG) that executes sequentially BEFORE the final Semantic Determinism schema validation.

The AVG MUST perform a semantic correlation check between the original isolated prompt ($P_i$) and the generated intent ($I_g$). Let $E(x)$ be a deterministic embedding function projecting semantic meaning into a continuous vector space $\mathbb{R}^n$.

The gate computes the cosine similarity distance $D_{sem}$:
$D_{sem} = 1 - \frac{E(P_i) \cdot E(I_g)}{||E(P_i)|| ||E(I_g)||}$

A strict rejection threshold $\tau$ MUST be defined by the host environment (typically $\tau < 0.15$ for highly constrained operations).

If $D_{sem} > \tau$, the intent MUST be rejected immediately as a potential prompt injection anomaly, raising a `Security_Violation_Anomaly`.

### 2. Guard Metadata Ledger Schema Update

To support the AVG, every payload evaluated MUST be logged, including the embedding vectors and the calculated distance, strictly adhering to the following schema extension.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "AdversarialGateLogEntry",
  "type": "object",
  "properties": {
    "intent_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$",
      "description": "Unique UUIDv4 for the intent evaluation."
    },
    "semantic_distance": {
      "type": "number",
      "description": "Calculated semantic divergence distance."
    },
    "threshold": {
      "type": "number",
      "description": "The configured strict rejection threshold."
    },
    "decision": {
      "type": "string",
      "enum": ["admit", "reject"],
      "description": "The final outcome of the AVG."
    }
  },
  "required": ["intent_id", "semantic_distance", "threshold", "decision"],
  "additionalProperties": false
}
```

### 3. Execution Sequence Diagram

The deterministic sequence of operations MUST follow this strict order.

```text
Host App                 AI Sandbox                 AVG Engine              Deterministic Control
   |                         |                          |                             |
   |---(1) Injects Prompt--->|                          |                             |
   |                         |                          |                             |
   |                         |---(2) Generates Intent-->|                             |
   |                         |                          |                             |
   |                         |                          |---(3) Computes D_sem        |
   |                         |                          |                             |
   |                         |                          |---(4) IF D_sem > tau : REJECT
   |                         |                          |                             |
   |                         |                          |---(5) ELSE : Admit Intent-->|
   |                         |                          |                             |
   |                         |                          |                             |---(6) Schema Validation
   |                         |                          |                             |
   |<=========================(7) Execute State Mutation==============================|
```

## Rationale

By implementing an adversarial gate based on semantic embedding correlation *prior* to strict JSON schema validation, we address the critical vulnerability where LLMs output semantically malicious data wrapped in perfectly valid deterministic schemas. By mathematically bounding the acceptable deviation from the core system prompt, prompt injection attempts are algorithmically neutralized.

## Backwards Compatibility

This is a forward-compatible extension to the SDIC-1 architecture. It introduces an optional but highly recommended pre-validation layer. It does not alter the underlying Action Ledger or Semantic Determinism schemas.
