[English](README.md) | [Français](README.fr.md)

# SDIC-1: Systemic Deterministic Integration of AI Capabilities

Author: Charles EDOU NZE
Status: Draft
Type: Standards Track
Created: 2026-05-25
---

## Abstract

This standard defines a strict software architecture that allows the integration of probabilistic cognitive capabilities (such as Large Language Models or LLMs) within deterministic enterprise applications. The primary goal of SDIC-1 is to remove the direct access of autonomous agents to persistence and execution layers, by introducing an isolation boundary and a standardized intent ledger.

## 1. Motivation

The rise of "AI-Native" architectures and "Vibe Coding" poses a systemic risk for highly regulated industries (Banking, Industry, Healthcare). A probabilistic model does not guarantee the repeatability of its output structure, which makes it incompatible with traditional software security and compliance requirements.

Similar to structured programming standards or decentralized protocols (EIP/ERC), SDIC-1 formalizes the indispensable security boundaries required to transform an unpredictable generative force into a controllable and auditable application component.

## 2. Specification

The protocol relies on the mandatory implementation of four layers of software abstractions independent of the programming language used (Agnostic Stack).

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
|   (Strict Schema Validation / Guardrails)                   |
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

### 2.1. Cognitive Isolation (The Sandbox Boundary)

The AI's execution environment (LLM, Agent) must be structurally isolated from the host infrastructure.

- **Golden Rule:** The AI possesses no network write permissions, no direct access to the main database, and no rights to call mutative APIs.
- **Interface Contract:** The AI acts exclusively as a semantic transformer. It receives a filtered textual stream and returns only a structured object representing an intent of action, without being able to execute it itself.

### 2.2. Semantic Determinism (Zero-Trust Schema Validation)

Any data outputting from the Cognitive Sandbox must be treated as unsafe user input (Untrusted Input).

- **Control Mechanism:** The host application must intercept the AI's payload and subject it to a strict schema validation (e.g., JSON Schema).
- **Deviation Management:** If the structure of the AI's response diverges from the expected schema (key change, incorrect data type, omission of a required field), the host application must immediately raise a software exception. The system must then fallback to a deterministic recovery process (Fallback) or require human arbitration.

### 2.3. The Action Ledger (Temporal Auditability)

Each validated action originating from an AI intent must be immutably recorded prior to its execution.
The action registry (Action Ledger) must save a snapshot containing the following data structures:

| Field | Type | Description |
|---|---|---|
| transaction_id | UUIDv4 | Unique and immutable identifier of the transaction. |
| timestamp | DateTime | Precise system-level timestamp of the interaction. |
| model_snapshot | String | Exact version of the model used (e.g., claude-3-5-sonnet, gpt-5-preview). |
| hyperparameters | Object | Runtime model configuration (Temperature, Top_P, Max Tokens). |
| cognitive_context | Text | Exact copy of the system prompt and data injected via RAG. |
| raw_response | Text | Raw response returned by the model prior to validation. |

### 2.4. Surgical Hydration (Context Efficiency Boundary)

In order to minimize the attack surface by prompt injection, latency, and operational costs, the context window sent to the AI must be restricted to the strict minimum necessary.

- **Mitigation Principle:** It is prohibited to transmit entire database tables or unfiltered complete histories.
- **Specification:** The host application must use vector indexing strategies (RAG) or application filtering to extract only the atom of data required for the resolution of the current task.

## 3. Rationale

The choice of a decoupled and asynchronous architecture rather than a native integration of execution agents (such as direct Function Calling or autonomous CLI agents) is motivated by the necessity to guarantee system governance.
By delegating the generation of intent to the AI and the validation/execution to traditional code, SDIC-1 enables senior engineers to retain absolute control over the company's business rules.

## 4. Security Considerations

- **Prompt Injection Mitigation:** Strict adherence to Pillar II (Semantic Determinism) guarantees that even if an attacker manages to manipulate the model via prompt injection, the generated output will be rejected by the schema validator if the intent does not match the authorized structures.
- **Data Leakage Prevention:** Adherence to Pillar IV limits the accidental exposure of sensitive enterprise data to third-party or shared models.
