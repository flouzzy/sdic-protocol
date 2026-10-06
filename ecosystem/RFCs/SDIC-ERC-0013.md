---
eip: 13
title: External Auditability and Quantified Safety Guarantees for Autonomous Agents and Reinforcement Learning in Critical Domains
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
category: Core
created: 2026-05-25
license: CC0-1.0
---

[English](SDIC-ERC-0013.md) | [Français](SDIC-ERC-0013.fr.md)

## Abstract

This standard defines a deterministic framework for certifying autonomous agent outputs and quantifying safety guarantees within critical domains such as medicine and robotics. It establishes an external auditing protocol and mathematical safety envelope for probabilistic models, including Reinforcement Learning (RL) agents. By coupling runtime verification with formal probabilistic risk bounds, SDIC-ERC-0013 enables third-party auditors to cryptographically verify that every executed action adheres strictly to predefined safety rules and quantified tolerance thresholds.

## Motivation

As autonomous agents and reinforcement learning systems transition into high-stakes environments—such as automated surgical assistance, pharmaceutical dosage calculation, and industrial robotics—probabilistic behavior presents severe liability and safety risks. Traditional testing methods fail to guarantee safety because autonomous policies adapt dynamically to complex state spaces.

To achieve safety certification in regulated domains, AI systems must provide two key capabilities:

1. **Quantified Safety Guarantees:** A rigorous, mathematically verifiable bound on the probability of rule violation ($P(\text{Violation}) \le \epsilon$).
2. **External Auditability:** An unalterable mechanism allowing independent external entities to audit agent intents, safety proofs, and execution histories without possessing internal model weights.

SDIC-ERC-0013 bridges the gap between probabilistic decision-making and deterministic safety guarantees by establishing formal audit schemas and runtime safety barriers.

## The Aviation Black Box and Flight Envelope Analogy

Imagine a commercial airliner equipped with an advanced automatic flight management system. The airplane operates within a strict physical flight envelope determined by structural integrity, stall speeds, and maximum aerodynamic loads.

To guarantee passenger safety, two distinct mechanisms exist: a real-time flight envelope protection controller and a tamper-proof flight data recorder (the black box). The flight controller monitors all pilot and autopilot commands. If an autopilot command attempts to pitch the nose up beyond the safe angle of attack, the physical protection system overrides the signal and maintains the aircraft within safe aerodynamic limits. Meanwhile, the flight data recorder continuously logs every control input, sensor reading, and system override with high-precision timestamps and cryptographic seals. After every flight, civil aviation authorities can extract this data to conduct independent external audits, verifying that safety boundaries were respected throughout the flight.

### Technical Mapping

- **The Airliner Flight Envelope:** The deterministic safety rules and boundary constraints (e.g., maximum drug dosage in medicine or joint torque limits in robotics).
- **The Autopilot System:** The autonomous AI agent or Reinforcement Learning policy generating probabilistic action proposals.
- **The Flight Envelope Protection Controller:** The SDIC Deterministic Control Layer enforcing zero-trust schema validation and safety barrier functions.
- **The Overridden Command:** An unsafe semantic intent rejected by the control layer when safety limits are breached.
- **The Flight Data Recorder (Black Box):** The Action Ledger storing cryptographic proofs of intent, state contexts, and execution logs.
- **The Civil Aviation Authority Audit:** The External Audit Protocol where third-party regulators evaluate verifiable audit certificates to validate safety compliance.

## Specification

### 1. Quantified Safety Envelope ($P(\text{Safety}) \ge 1 - \epsilon$)

Autonomous agents operating under SDIC-ERC-0013 MUST explicitly quantify their safety guarantees for any proposed intent. For Reinforcement Learning policies operating over Constrained Markov Decision Processes (CMDPs), the agent emits an intent containing an estimated failure probability $\epsilon$ alongside its action proposal.

Let $S$ be the current system state, $A$ be the proposed action, and $C(S, A)$ be the cost function representing safety rule violation. The intent is valid if and only if:

$$P(C(S, A) > 0 \mid S) \le \epsilon_{\text{threshold}}$$

where $\epsilon_{\text{threshold}}$ is a deterministic, domain-specific invariant enforced by the host application.

### 2. Domain-Specific Safety Schemas

#### 2.1. Medical Assistance & Pharmacological Intent Schema

Medical AI agents MUST submit intents complying with strict pharmacological dosage bounds and clinical protocol parameters. All JSON Schemas enforce `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MedicalActionIntent",
  "type": "object",
  "properties": {
    "patient_id": {
      "type": "string",
      "pattern": "^PAT-[0-9]{8}$"
    },
    "protocol_id": {
      "type": "string",
      "pattern": "^PROTO-[A-Z0-9]{6}$"
    },
    "medication_code": {
      "type": "string",
      "pattern": "^Rx-[0-9]{6}$"
    },
    "dosage": {
      "type": "object",
      "properties": {
        "value": {
          "type": "number",
          "minimum": 0.01,
          "maximum": 500.00
        },
        "unit": {
          "type": "string",
          "enum": ["mg", "ml", "mcg", "IU"]
        }
      },
      "required": ["value", "unit"],
      "additionalProperties": false
    },
    "estimated_risk_epsilon": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 0.001
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "signature": {
      "type": "string"
    }
  },
  "required": [
    "patient_id",
    "protocol_id",
    "medication_code",
    "dosage",
    "estimated_risk_epsilon",
    "timestamp",
    "signature"
  ],
  "additionalProperties": false
}
```

#### 2.2. Robotic Actuation & Kinematic Safety Schema

Robotic control agents (e.g., surgical or industrial manipulators) MUST emit action proposals containing target joint velocities, Cartesian boundary coordinates, and torque limits.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RoboticActuationIntent",
  "type": "object",
  "properties": {
    "robot_id": {
      "type": "string",
      "pattern": "^ROB-[0-9]{6}$"
    },
    "command_type": {
      "type": "string",
      "enum": ["joint_trajectory", "cartesian_pose", "emergency_stop"]
    },
    "joint_velocities_rad_s": {
      "type": "array",
      "items": {
        "type": "number",
        "minimum": -3.14159,
        "maximum": 3.14159
      },
      "minItems": 6,
      "maxItems": 6
    },
    "max_torque_nm": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 150.0
    },
    "collision_envelope_clearance_mm": {
      "type": "number",
      "minimum": 10.0
    },
    "estimated_risk_epsilon": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 0.0001
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "signature": {
      "type": "string"
    }
  },
  "required": [
    "robot_id",
    "command_type",
    "joint_velocities_rad_s",
    "max_torque_nm",
    "collision_envelope_clearance_mm",
    "estimated_risk_epsilon",
    "timestamp",
    "signature"
  ],
  "additionalProperties": false
}
```

### 3. External Audit Certificate Schema

External auditors verify compliance by reading immutable Action Ledger entries and evaluating cryptographic audit certificates. The host application or external auditor generates an `AuditCertificate` after verifying the canonical intent signature and safety invariants.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "VerifiableAuditCertificate",
  "type": "object",
  "properties": {
    "audit_id": {
      "type": "string",
      "format": "uuid"
    },
    "transaction_id": {
      "type": "string",
      "format": "uuid"
    },
    "auditor_identity": {
      "type": "string"
    },
    "verification_result": {
      "type": "string",
      "enum": ["PASSED", "FAILED_SCHEMA", "FAILED_SAFETY_INVARIANT", "FAILED_SIGNATURE"]
    },
    "evaluated_epsilon": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0
    },
    "canonical_intent_hash": {
      "type": "string",
      "pattern": "^[a-fA-F0-9]{64}$"
    },
    "auditor_signature": {
      "type": "string"
    }
  },
  "required": [
    "audit_id",
    "transaction_id",
    "auditor_identity",
    "verification_result",
    "evaluated_epsilon",
    "canonical_intent_hash",
    "auditor_signature"
  ],
  "additionalProperties": false
}
```

### 4. Cryptographic Proof of Intent and Non-Repudiation

To ensure that intent payload tampering is impossible, the agent signs a canonical representation of the intent excluding the `signature` field:

1. **Stripping:** Strip the `"signature"` field from the JSON object.
2. **Canonicalization:** Apply JSON Canonicalization Scheme (RFC 8785) sorting keys lexicographically and eliminating whitespace.
3. **Hashing:** Compute SHA-256 over the canonical string.
4. **Signing:** Sign the SHA-256 hash using the agent's private key (e.g., Ed25519).
5. **Validation:** The Deterministic Control Layer and External Auditor verify the signature against the agent's public key before execution or certification.

### 5. Interaction Sequence Diagram

```text
+----------+              +-----------------------+              +---------------+              +----------------+
| AI Agent |              | Deterministic Control |              | Action Ledger |              | External Audit |
+----------+              +-----------------------+              +---------------+              +----------------+
     |                                |                                  |                               |
     | 1. Submit Intent (Epsilon)     |                                  |                               |
     |------------------------------->|                                  |                               |
     |                                | 2. Validate Schema               |                               |
     |                                |    & Safety Bounds               |                               |
     |                                |--------------------------        |                               |
     |                                |                         |        |                               |
     |                                |<-------------------------        |                               |
     |                                |                                  |                               |
     |                                | 3. Verify Signature & Commit     |                               |
     |                                |--------------------------------->|                               |
     |                                |                                  |                               |
     |                                |                                  | 4. Fetch Ledger History       |
     |                                |                                  |<------------------------------|
     |                                |                                  |                               |
     |                                |                                  | 5. Generate Audit Certificate |
     |                                |                                  |------------------------------>|
     |                                |                                  |                               |
```

## Security Considerations

1. **Quantified Bound Spoofing:** A compromised agent might report a falsely low risk estimation $\epsilon$. The Deterministic Control Layer MUST NOT rely solely on the agent's self-reported $\epsilon$; it MUST execute independent, deterministic safety checks against physical/medical invariants.
2. **Replay and Relabeling Attacks:** Validating signatures over the RFC 8785 canonical hash prevents malicious actors from altering non-functional JSON formatting or reusing signed intents across different transactions.
3. **Audit Ledger Tampering:** External auditability depends on Action Ledger immutability. Ledgers should use cryptographic hash chains (Merkle trees) to prevent retrospective history rewriting.

## Backward Compatibility

SDIC-ERC-0013 is fully compatible with SDIC-1. It extends the Semantic Determinism layer by embedding domain-specific bounds and formal risk quantifiers into standard intent payloads without altering core protocol state transitions.

## Copyright

Copyright and related rights waived via CC0.
