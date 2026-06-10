---
eip: 3
title: PRISM - A DNS-Inspired Semantic Routing Architecture for Specialized Large Language Models
author: Mossaab Hassana Souaissa <mossaab.souaissa@gmail.com>
status: Draft
type: Standards Track
category: Core
created: 2026-06-10
---

[English](SDIC-RFC-0003.md) | [Français](SDIC-RFC-0003.fr.md)

## Abstract

The industrial deployment of Large Language Models (LLMs) reveals structural limitations linked to their centralized architecture, particularly regarding response legitimacy, accountability, cost, and latency. Current systems produce a response even when a query exceeds their true scope of expertise, lacking an explicit refusal or prior routing mechanism.

This paper proposes an architectural framework based on a distributed infrastructure of specialized models, coordinated by a semantic routing mechanism inspired by DNS. The central hypothesis is that knowledge routing must precede its resolution.

We introduce a binding framing protocol, called 153-key, enabling the qualification, orientation, and self-assessment of models, as well as a dynamic skills directory and an empirical feedback loop. A formal state machine strictly separates the legitimacy decision from business execution.

## Motivation

Current centralized LLM architectures present several structural limitations: the production of responses outside the actual scope of competence, the absence of a native refusal mechanism, difficulty in clarifying the legitimacy of a response, high costs associated with the systematic use of generalist models, and latency levels incompatible with certain industrial use cases.

The proposed architecture, PRISM (Protocol for Routed Intelligent Specialized Models), addresses these limitations by routing a query to the competent entity before any resolution occurs, explicitly separating decision from execution. Knowledge must be routed before it is resolved. An AI system should not attempt to respond until the questions of legitimacy, scope, and competence level have been explicitly addressed.

## The International Postal Sorting Facility Analogy

Imagine an international postal sorting facility handling millions of packages daily. Packages arrive with various addresses, sizes, and contents. If every worker tried to open every package, process its contents, and deliver it themselves, the system would collapse into chaos, leading to misdeliveries and immense delays. Instead, the facility operates with strict routing protocols before any delivery is attempted.

When a package enters the facility, it first encounters a preliminary scanning station that only reads the zip code—it does not open the package. This station consults a dynamic directory of regional delivery hubs. If the package belongs to a specialized region, it is routed to that specific hub. Each hub has strict rules about what it can handle (e.g., a hub for fragile items refuses heavy machinery). Only when a package is confirmed to be at the correct, legitimate hub is it opened and processed for final delivery.

In our PRISM architecture:

- **The Incoming Packages** represent user queries, which are unstructured and cover a multitude of domains.
- **The Preliminary Scanning Station** represents the Central Routing Console (VSLM). It analyzes the semantic intent of the query without attempting to answer it, acting as a pre-resolver.
- **The Dynamic Directory of Hubs** maps to the 153-Registry, a living database referencing all available models and their explicit capabilities.
- **The Strict Rules of the Hubs** embody the 153-key protocol, which enforces the qualification of models and mandates explicit refusal if a query falls outside their domain.
- **The Final Processing for Delivery** is the business execution phase, which only occurs after the routing decision is formally validated by the state machine.

## Specification

### 1. Central Routing Console (VSLM)

The central console is implemented as a Very Small Language Model (VSLM) constrained to perform intent classification only. It acts as a pre-resolver, analogous to a DNS server, analyzing the intent of the query, consulting the skills directory, and making the routing decision.

```text
+----------+          +-------------+          +--------------+
|   User   |          |    VSLM     |          | 153-Registry |
+----+-----+          +------+------+          +-------+------+
     |                       |                         |
     | 1. Query              |                         |
     +---------------------->|                         |
     |                       | 2. Analyze Intent       |
     |                       +------------------------>|
     |                       |                         | 3. Return Model ID
     |                       |<------------------------+
     |                       |                         |
     |                       | 4. Route Query          |
     |                       +--------------------------------> [Specialized Model]
```

### 2. Dynamic Skills Directory (153-Registry)

The directory is a living database referencing all available models. Each entry is described by a Capability Card. No model is presumed to be an expert by default.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RegistryEntry",
  "type": "object",
  "properties": {
    "model_id": {
      "type": "string",
      "description": "Unique identifier for the specialized model."
    },
    "domain": {
      "type": "string",
      "description": "The primary domain of expertise."
    },
    "sub_domains": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Specific sub-domains the model handles."
    },
    "competence_score": {
      "type": "number",
      "minimum": 0,
      "maximum": 100,
      "description": "Empirical competence score."
    },
    "average_latency_ms": {
      "type": "number",
      "description": "Average response latency in milliseconds."
    },
    "refusal_rate": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Rate at which the model appropriately refuses queries."
    },
    "status": {
      "type": "string",
      "enum": ["active", "inactive", "banned"],
      "description": "Current status in the directory."
    },
    "last_update": {
      "type": "string",
      "format": "date",
      "description": "Date of the last capability update."
    }
  },
  "required": [
    "model_id",
    "domain",
    "sub_domains",
    "competence_score",
    "average_latency_ms",
    "refusal_rate",
    "status",
    "last_update"
  ],
  "additionalProperties": false
}
```

### 3. The 153-key: Qualification and Execution

The 153-key is a binding framing protocol enforcing qualification and execution. The formal state machine strictly separates the legitimacy decision phase from the business execution phase.

```text
+-------------+          +-------------------+          +----------------+
|    VSLM     |          | Specialized Model |          | Business Logic |
+------+------+          +---------+---------+          +-------+--------+
       |                           |                            |
       | 1. Routed Query           |                            |
       +-------------------------->|                            |
       |                           | 2. Evaluate Domain Match   |
       |                           |------------------+         |
       |                           |                  |         |
       |                           |<-----------------+         |
       |                           |                            |
       | 3a. REFUSED (Out of Scope)|                            |
       |<--------------------------+                            |
       |                           |                            |
       |                           | 3b. ACCEPTED (In Scope)    |
       |                           +--------------------------->|
```

Every execution response MUST adhere to the following schema:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Protocol153KeyResponse",
  "type": "object",
  "properties": {
    "status": {
      "type": "string",
      "enum": ["ACCEPTED", "REFUSED", "REJECTED"],
      "description": "The decision status for the query."
    },
    "reason": {
      "type": "string",
      "maxLength": 100,
      "description": "Justification for the acceptance or refusal."
    },
    "response": {
      "type": "string",
      "description": "The technical response if accepted, or empty if refused."
    }
  },
  "required": [
    "status",
    "reason",
    "response"
  ],
  "additionalProperties": false
}
```

## Backward Compatibility

This RFC extends the SDIC-1 framework without breaking existing deterministic isolation boundaries.
