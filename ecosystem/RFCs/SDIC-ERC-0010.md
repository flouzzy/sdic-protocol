---
title: "SDIC-ERC-0010: Standardized Runtime Prompt Injection Filtering"
author: "Jules <jules@example.com>"
status: Draft
type: Standards Track
created: 2026-08-01
---

[English](SDIC-ERC-0010.md) | [Français](SDIC-ERC-0010.fr.md)

## Abstract

This standard proposes an architectural pattern for mitigating prompt injection attacks at the runtime level within the SDIC-1 framework. It aims to enhance Pilier I (Cognitive Isolation) by introducing a determinist semantic firewall that sanitizes injected context before it reaches the AI sandbox.

## 1. Introduction

To understand the Runtime Semantic Firewall, imagine a state-of-the-art municipal water filtration plant. The city's reservoir collects water from various sources, including unpredictable natural rainfall and potentially contaminated runoff.

If this raw, unfiltered water were pumped directly into the pristine homes of the citizens, the entire water system could become corrupted, leading to widespread illness. Therefore, before the water ever touches the city's main distribution network, it must pass through a strict, physical filtration membrane. This membrane is not intelligent; it is purely mechanical. It allows water molecules to pass through while physically blocking any particulate matter larger than a specific, rigidly defined size.

In the SDIC-1 architecture:

- **The Raw Water:** This represents the user-provided inputs or external API responses that are potentially laden with malicious prompt injection attempts.
- **The Citizens' Homes:** This is the AI Sandbox (The Cognitive Model). If malicious instructions reach the model unhindered, it might bypass its initial constraints.
- **The Physical Filtration Membrane:** This is the Runtime Semantic Firewall. It does not attempt to "understand" the text. Instead, it enforces strict structural and deterministic rules (like removing unauthorized delimiters or validating input types) to strip away malicious payloads before they can interact with the system prompt.

## 2. Rationale & Architectural Impact

As models become more capable, the surface area for prompt injection via manipulated user input increases. Relying solely on the model's internal alignment or the final JSON schema validation (Pilier II) is insufficient; if a model is fully hijacked, it could generate adversarial outputs designed to exploit vulnerabilities in the parsing layer itself.

The Runtime Semantic Firewall introduces an essential preemptive security layer. By deterministically sanitizing the context *before* it is injected into the AI's prompt, we drastically reduce the probability of successful cognitive hijacking.

## 3. Proposed Specification Changes

### 3.1. Target Component

This upgrade specifically targets **Pilier I: Cognitive Isolation**, establishing a mandatory pre-processing step before the filtered context reaches the AI Sandbox.

### 3.2. Context Sanitization Rule

Before any external data (user input, external API payload) is concatenated with the system prompt, it must be validated against a strict schema to ensure it contains no executable code or instruction-override delimiters.

### 3.3. Standardized Input Schema

The host application MUST enforce the following JSON Schema on all untrusted input meant for the cognitive context.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SanitizedContextInput",
  "type": "object",
  "properties": {
    "source": {
      "type": "string",
      "enum": ["user_input", "external_api"]
    },
    "raw_payload": {
      "type": "string",
      "maxLength": 4096,
      "pattern": "^[^<>{}\\[\\]]*$"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    }
  },
  "required": ["source", "raw_payload", "timestamp"],
  "additionalProperties": false
}
```

*Note: The regex pattern `^[^<>{}\\[\\]]*$` explicitly forbids common formatting and structural characters used in prompt injection (XML tags, JSON brackets) within the raw payload.*

### 3.4. Sequence Diagram

```text
+-------------+                 +-----------------------------+               +----------------+
| Untrusted   |                 | Runtime Semantic Firewall   |               | AI / LLM       |
| Source      |                 | (Host Application)          |               | Sandbox        |
+-------------+                 +-----------------------------+               +----------------+
      |                                       |                                       |
      | 1. Submit Data                        |                                       |
      |-------------------------------------->|                                       |
      |                                       | 2. Schema & Regex Validation          |
      |                                       |-----------------------------          |
      |                                       |                            |          |
      |                                       |<----------------------------          |
      |                                       |                                       |
      |                                       | 3. Reject if Invalid                  |
      |<--------------------------------------| (Drop Request)                        |
      |                                       |                                       |
      |                                       | 4. Inject Sanitized Context           |
      |                                       |-------------------------------------->|
      |                                       |                                       |
```

## 4. Backward Compatibility

This specification is fully backward compatible. It acts as an optional (but highly recommended) pre-processing step for the host application and does not alter the core Action Ledger or final Semantic Determinism layers defined in v1.0.0-draft.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
