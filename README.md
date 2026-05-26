[English](README.md) | [Français](README.fr.md)

# SDIC-1: Systemic Deterministic Integration of AI Capabilities

Author: Charles EDOU NZE
Status: Draft
Type: Standards Track
Created: 2026-05-25
---

## Abstract

This standard defines a strict software architecture that integrates probabilistic cognitive capabilities (Large Language Models, autonomous agents) into deterministic enterprise applications. The primary goal of SDIC-1 is to eradicate the direct access of autonomous agents to persistence and execution layers by introducing a mathematical boundary of isolation, cryptographic non-repudiation, and a standardized intent ledger.

## 1. Introduction: The Feynman Analogy

To understand SDIC-1, imagine a nuclear power plant. The AI (the Large Language Model) is the radioactive core: it is immensely powerful, capable of generating massive amounts of energy (intelligence), but fundamentally chaotic, probabilistic, and dangerous if exposed directly to the outside world.

The traditional host application is the concrete containment building and the control rods. We never expose the city directly to the radioactive core. Instead, we capture the heat (the **semantic intent**), channel it through highly resilient, strictly standardized pipes (the **JSON Schema Validator**), and use that controlled pressure to turn a turbine (the **deterministic execution**).

SDIC-1 is the exact specification for building those pipes, ensuring that an unpredictable generative force can safely power highly regulated, deterministic machinery.

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
