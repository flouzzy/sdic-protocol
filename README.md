[English](README.md) | [Français](README.fr.md)

# SDIC-1: Systemic Deterministic Integration of AI Capabilities

- Author: Charles EDOU NZE
- Status: Draft
- Type: Standards Track
  
Created: 2026-05-25
---

## Abstract

This standard defines a strict software architecture that integrates probabilistic cognitive capabilities (Large Language Models, autonomous agents) into deterministic enterprise applications. The primary goal of SDIC-1 is to eradicate the direct access of autonomous agents to persistence and execution layers by introducing a mathematical boundary of isolation, cryptographic non-repudiation, and a standardized intent ledger.

## 1. Introduction

To understand SDIC-1, imagine a nuclear power plant. The radioactive core is immensely powerful, capable of generating massive amounts of energy, but it is fundamentally chaotic, probabilistic, and dangerous if exposed directly to the outside world.

The concrete containment building and the control rods encase this core. We never expose the city directly to the radiation. Instead, we capture the generated heat, channel it through highly resilient, strictly standardized pipes, and use that controlled pressure to turn a turbine.

SDIC-1 is the exact specification for building those pipes, ensuring that an unpredictable generative force can safely power highly regulated, deterministic machinery. In this architecture, each component of the analogy corresponds to a precise and rigorous element of the system:

### The Radioactive Core: Artificial Intelligence (The Probabilistic Model)

This represents the cognitive engine (Large Language Models or autonomous agents). Just as the nuclear core produces energy chaotically, the AI generates understanding and data through massive probabilistic calculations. Its outputs are powerful but non-deterministic. This is why the core must have absolutely no direct access to critical control mechanisms or system databases.

### The Containment Building and Control Rods: The Host Application

This is the classic deterministic system (such as a business application built in Symfony, React, Go, or Java). Its role is to encapsulate the AI's power, provide it with a sterile execution environment (Sandbox), and modulate its activity via strict contexts. The host ensures complete isolation and prevents any corruption of the system state by unmanaged operations.

### The Captured Heat: The Semantic Intent

The heat represents the action proposed by the AI. Instead of modifying the state directly, the AI expresses what it "wishes" to accomplish in the form of data. It is this raw willpower, extracted from the model, that constitutes the informational energy of the system. It is the abstract bridge between context comprehension and action execution.

### The Resilient Pipes: The JSON Schema Validator (Semantic Determinism)

These pipes embody the mathematical and control boundary. They receive the raw semantic intent and intractably verify that it perfectly matches a cryptographically and structurally defined JSON Schema. Any intent (heat) that does not conform to this strict canonical form is rejected or dissipated, ensuring that only compliant and secure information reaches the turbines.

### The Turbine: Deterministic Execution (The State Transition Engine)

The turbine transforms the regulated pressure into predictable and useful action. Once the semantic intent has successfully passed the deterministic filter of the pipes, it is transmitted to the execution engine (the Action Ledger and business logic). It is only here that the system state is mutated, with absolute mathematical certainty, thereby isolating persistence from the probabilistic universe.

## 2. Mathematical Formalism of Cognitive Isolation

The risk of "AI-Native" architectures lies in state degradation. Let $S$ be the deterministic state of the host system, and $C$ be the cognitive capability of the probabilistic model.

Allowing an AI to directly transition the system state yields a probabilistic outcome:
$P(S_{t+1} | S_t, C) < 1$ (The state transition is non-deterministic).

SDIC-1 forces a strict separation. The model may only compute a structured Intent $I$ based on a filtered context $X$:
$I = C(X)$

A deterministic validation function $V(I)$ acts as an absolute binary gate. Let $\Phi$ be the strict expected schema:
$$
V(I) =
\begin{cases}
1, & \text{if } I \equiv \Phi \\
0, & \text{otherwise}
\end{cases}
$$

The state transition is subsequently handled purely by the deterministic execution engine $E$:
$S_{t+1} = E(S_t, I)$ iff $V(I) = 1$.
If $V(I) = 0$, the system state is preserved ($S_{t+1} = S_t$) and an exception is raised.

## 3. Specification: The Agnostic Stack

The protocol relies on the mandatory implementation of four agnostic layers.

```text
+-------------------------------------------------------------+
|                      Host Application                       |
|   (Deterministic System: Symfony, React, Go, etc.)          |
+------------------------------+------------------------------+
                               |
                 [1. Filtered Context & Prompt]
                               |
                               v
+-------------------------------------------------------------+
|                     AI / LLM Sandbox                        |
|   (Cognitive Isolation - Analysis & Intent Emission)        |
+------------------------------+------------------------------+
                               |
                    [2. Raw JSON Intent]
                               |
                               v
+-------------------------------------------------------------+
|                Deterministic Control Layer                  |
|   (Strict Schema Validation / Cryptographic Verification)   |
+------------------------------+------------------------------+
                               |
                  [3. Validated Intent / Rejection]
                               |
                               v
+-------------------------------------------------------------+
|                        Action Ledger                        |
|   (Immutable Registry of Persistence and Execution)         |
+-------------------------------------------------------------+
```

### 3.1. Cognitive Isolation (The Sandbox Boundary)

The AI's execution environment must be structurally isolated.

- **Golden Rule:** The AI possesses no network write permissions, no direct database access, and no rights to invoke mutative APIs.
- **Interface Contract:** The AI acts exclusively as a semantic transformer. It emits an intent object, incapable of executing it itself.

### 3.2. Semantic Determinism (Zero-Trust Schema Validation)

All outputs from the Sandbox must be treated as `Untrusted Input`. The validation layer must enforce strict schema adherence.
When defining JSON Schemas within the repository, strictness must be enforced by setting `"additionalProperties": false` at the top level and on applicable nested objects to prevent unspecified key injections.

**Example Strict Schema Definition:**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TransferIntent",
  "type": "object",
  "properties": {
    "action": { "type": "string", "enum": ["transfer_funds"] },
    "target_id": { "type": "string", "pattern": "^[0-9a-fA-F-]{36}$" },
    "justification": { "type": "string" },
    "signature": { "type": "string" }
  },
  "required": ["action", "target_id", "justification", "signature"],
  "additionalProperties": false
}
```

### 3.3. Cryptographic Identity & Proof of Intent

For fully autonomous agent operations, an intent must be non-repudiable. To prevent replay and relabeling attacks, the Action Ledger requires a cryptographic signature.
The agent must generate a signature $\sigma$ over the canonical representation of the entire entry (excluding the signature field itself).

$\sigma = Sign_{AgentPrivKey}(Hash(Canonical(I)))$

The host application must validate $\sigma$ against the agent's known Public Key prior to execution.

### 3.4. The Action Ledger (Temporal Auditability)

Each validated action must be immutably recorded before execution.

| Field | Type | Description |
|---|---|---|
| transaction_id | UUIDv4 | Unique, immutable identifier of the transaction. |
| timestamp | DateTime | System-level precise temporal marker. |
| model_snapshot | String | Exact version of the model (e.g., claude-3-5-sonnet). |
| hyperparameters | Object | Runtime model configuration (Temperature, Top_P). |
| cognitive_context | Text | Exact copy of the system prompt and injected data. |
| raw_response | Text | Unaltered raw response returned by the model. |
| proof_of_intent | String | Cryptographic signature of the canonical intent. |

### 3.5. Continuous Memory & Swarm Consensus

To enable persistence of consciousness (reminiscent of autonomous digital minds), the AI cannot store state internally. State is maintained in the Action Ledger and deterministically "re-hydrated" into the AI's context window on each invocation.

For multi-agent systems (Swarm Consensus), agents may not mutate shared state directly. They must emit $Intent_{Proposal}$ to a shared deterministic ledger, which triggers a consensus validation function before committing the state.

## 4. Security Considerations

- **Prompt Injection Mitigation:** Strict Semantic Determinism guarantees that even if a model is successfully hijacked via prompt injection, any output diverging from the strict JSON Schema will be deterministically rejected by the host.
- **Relabeling Attacks:** Validating the signature over the *canonical representation* prevents attackers from subtly modifying the payload in transit.
