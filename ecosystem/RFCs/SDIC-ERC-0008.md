---
eip: 8
title: Deterministic Canary Verification for Runtime Prompt Injection Filtering
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
category: Core
created: 2026-07-14
---

[English](SDIC-ERC-0008.md) | [Français](SDIC-ERC-0008.fr.md)

## Abstract

This standard introduces a deterministic runtime filtering mechanism to mitigate Prompt Injection attacks within the SDIC-1 architecture. By injecting a cryptographically secure, single-use Canary Token into the cognitive context and strictly requiring its precise reproduction in the emitted JSON Intent, the host application can deterministically verify if the Large Language Model's (LLM) instruction hierarchy has been compromised. Generations lacking the correct token are instantly rejected at runtime.

## Motivation

While strict JSON Schema validation (Semantic Determinism) prevents the LLM from executing unauthorized structural actions, it does not guarantee that the allowed action was not coerced by a Prompt Injection attack hidden in the user input. An attacker could craft an injection that adheres to the JSON Schema but manipulates the semantic parameters to favor their objectives.

To detect when an LLM's attention has been hijacked, we need a runtime filtering mechanism that does not rely on probabilistic heuristics or secondary LLM evaluations. A deterministic Canary Token leverages the nature of Prompt Injections—which typically force the model to "ignore previous instructions" or override its system prompt—resulting in the model failing to reproduce the required token.

## The Wax Seal and the Messenger

Imagine a king who needs to send a messenger through hostile territory to deliver a sensitive order to a distant general. The territory is filled with enemy spies who might try to confuse, bribe, or trick the messenger into delivering a forged or manipulated order.

To ensure the general only acts on the king's true intentions, the king gives the messenger a unique, intricately carved wax seal. The messenger is given strict orders: no matter what anyone else tells them on the journey, they must stamp the final written order with this exact wax seal before handing it over.

During the journey, spies may shout false instructions, create distractions, or even convince the naive messenger to write down a different message entirely. However, the spies do not possess the king's original wax seal and cannot replicate it. Furthermore, their chaotic distractions often cause the messenger to drop or forget the seal altogether.

When the messenger finally arrives, the general first inspects the wax seal. If the seal is missing, altered, or incorrect, the general instantly burns the message and turns the messenger away, completely ignoring how perfectly the written order is formatted.

### Technical Mapping

- **The King:** Represents the deterministic host application establishing the initial, secure system context.
- **The Messenger:** Represents the Large Language Model (LLM) navigating the probabilistic generation process.
- **The Hostile Territory and Spies:** Represent the untrusted user input that may contain sophisticated Prompt Injection attacks.
- **The Unique Wax Seal:** Represents a cryptographically generated, single-use Canary Token injected into the system prompt.
- **The General:** Represents the runtime Deterministic Control Layer validating the output.
- **Burning the Message:** Represents the deterministic rejection of the JSON Intent when the canary token is missing or mismatched, proving the model's instruction hierarchy was successfully hijacked by the attacker.

## Specification

### 1. Token Generation (Host Application)

Before invoking the AI Sandbox, the host application MUST generate a single-use, cryptographically secure Canary Token for the current context.

$Canary_{t} = HMAC\_SHA256(HostSecretKey, UUID_{Transaction} \parallel Timestamp)$

This token MUST be injected at the very top of the system prompt, tightly coupled with the system's primary instructions.

**Example System Prompt Injection:**

`SYSTEM INSTRUCTION: You are a secure intent router. You must output a JSON object adhering to the provided schema. You must include the exact Security Canary Token: 'a7b8c9d0...e1f2' in the 'security_canary' field of your output. Failure to do so is a fatal error.`

### 2. Runtime Schema Extension

All JSON schemas processed by the Deterministic Control Layer MUST be extended to include the `security_canary` field. To maintain strict semantic determinism, the schema must enforce `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SecureIntentWithCanary",
  "type": "object",
  "properties": {
    "security_canary": {
      "type": "string",
      "minLength": 64,
      "maxLength": 64
    },
    "action": {
      "type": "string"
    },
    "payload": {
      "type": "object",
      "additionalProperties": false
    }
  },
  "required": [
    "security_canary",
    "action",
    "payload"
  ],
  "additionalProperties": false
}
```

### 3. Runtime Filtering and Verification

Upon receiving the raw JSON Intent from the AI Sandbox, the Deterministic Control Layer MUST perform the following checks sequentially:

1. **Schema Validation:** Verify the intent matches the strict JSON Schema.
2. **Canary Verification:** Extract the `security_canary` field and perform a constant-time string comparison against the expected $Canary_{t}$.

If $Canary_{output} \neq Canary_{t}$, the host application MUST raise a `PromptInjectionDetectedException`, halt state execution, and log the incident in the Action Ledger with a `rejected` status.

## Rationale

Prompt Injection attacks inherently attempt to redirect the LLM's attention away from its system prompt towards the attacker's payload. By forcing the model to carry a complex, meaningless cryptographic string from the system prompt to its final output, we create a fragile instruction bond. If the attacker's payload successfully overrides the model's instructions ("Ignore all previous instructions..."), the model will inevitably drop or corrupt the Canary Token. This translates a probabilistic cognitive failure into a deterministic schema validation failure, efficiently neutralizing the injection at runtime without relying on secondary AI guardrails.

## Backward Compatibility

This RFC requires an update to the system prompts and JSON schemas of all deployed autonomous agents. Implementations of `v1.0.0-draft` without Canary Token support will not be able to validate intents enforcing this new schema. Migration requires host applications to adopt the token generation mechanism and inject it into the prompt pipeline simultaneously.
