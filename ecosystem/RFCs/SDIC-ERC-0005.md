[English](SDIC-ERC-0005.md) | [Français](SDIC-ERC-0005.fr.md)

---

eip: 5
title: The 8 Stages of AI Engineering Maturity Framework
author: SDIC-1 Architecture Group
type: Standards Track
category: ERC
status: Draft
created: 2026-06-09
---

## Abstract

This ERC defines the 8 Stages of AI Engineering Maturity, providing a deterministic framework for evaluating and transitioning an organization's AI capabilities. It standardizes the progression from unmanaged, probabilistic individual usage to fully autonomous, infrastructure-hosted, and self-evaluating AI factories. This specification introduces a structural maturity model designed to align the lifecycle of AI development with strict governance, deterministic testing, and contextual orchestration.

## Motivation

As organizations integrate Large Language Models and autonomous agents into their software development lifecycles (SDLC), they frequently experience highly uneven adoption. As originally articulated by Fabien Potencier in his reflections on AI engineering maturity, an organization does not exist at a single point of capability; rather, it exhibits a distribution of maturity across different teams. This asymmetry creates integration friction, unpredictable computational expenditures, and security vulnerabilities.

Establishing a standardized, deterministic pathway—from isolated local execution to fully autonomous, infrastructure-level agent orchestration—allows organizations to systematically upgrade their engineering workflows without sacrificing quality or determinism.

## Analogy: The Municipal Water Grid

Consider the evolution of a settlement's water supply. Initially, residents walk to a river and fill individual buckets whenever they are thirsty. The process is chaotic, unmeasured, and entirely dependent on individual effort. Soon, people begin digging their own personal wells and building ad-hoc pumps in their backyards. Eventually, small neighborhoods pool their resources to build local water towers, but these remain disconnected from the rest of the town.

Recognizing the inefficiency, a central municipality steps in. They establish universal standards for pipe dimensions, water purity, and central metering. The town's workflow changes: instead of manually fetching water, residents now rely on deterministic pressure sensors and automated valves. The system evolves into an interconnected grid that optimizes flow based on strict mathematical constraints. Pumping stations become highly advanced and self-regulating, though they still require human supervisors on-site. Finally, the entire grid achieves full autonomy—it automatically detects leaks, reroutes pressure, schedules its own maintenance during off-peak hours, and guarantees purity without any human intervention.

### Technical Mapping

* **The River / Buckets:** Unmanaged AI endpoints (e.g., standard chat interfaces) where developers blindly paste code without shared context or configuration.
* **Personal Wells:** Individual developers optimizing their local environments with personal `AGENTS.md` files and isolated prompts.
* **Local Water Towers:** Isolated teams sharing context and skills, increasing their localized velocity while diverging from organizational standards.
* **Standards & Metering:** The implementation of enterprise governance, SSO/SCIM, token budgets, and central curation of models and context.
* **Pressure Sensors & Valves:** Process overhaul enforcing specification-first development, automated pull request (PR) reviews, and risk-based evaluations.
* **Interconnected Grid:** A unified system where Multi-Agent orchestration, shared Model Context Protocol (MCP) servers, and Test-Driven Development (TDD) act as the foundational dependencies.
* **Self-Regulating Stations:** High-autonomy agents running locally on developer machines, capable of generating and delivering complete units of work under human supervision.
* **Autonomous Grid:** Infrastructure-hosted agents running in scheduled, sandboxed environments with central observability, executing cyclic maintenance and feature deployments autonomously.

## Specification

The framework defines 8 sequential stages. Progressing to a subsequent stage mandates the fulfillment of the preceding stage's deterministic criteria.

### Stage 1: The Void

The organization lacks a defined governance strategy. Usage is characterized by the absence of guardrails, shared context files, or managed API keys. Developers manually interface with models to build an intuition for probabilistic outputs.

### Stage 2: The Drift

Divergence begins at the individual level. Specific engineers implement local optimizations, utilizing personal repositories of prompts, custom skills, and localized `AGENTS.md` files to automate their workflows. This stage represents the limit of zero-cost inaction; further divergence introduces structural risks.

### Stage 3: The Islands

The tooling delta expands from individuals to entire teams. Certain teams aggregate their `AGENTS.md` configurations, connect MCP servers, and develop reusable skills, drastically increasing their output velocity. This creates localized monopolies of efficiency and structural friction with legacy teams.

### Stage 4: The Standardization Wager

Leadership enforces explicit capability development. Context engineering is formalized into centralized repositories. Mandatory governance is introduced: SSO/SCIM integration, automated secret scanning, agent-result PR controls, and unified token budgeting.

### Stage 5: The Process Overhaul

The execution paradigm shifts. The SDLC enforces a "specification-first" architecture where the standard of entry is a deterministic specification housed in the repository. Code review transitions from syntax validation to risk-based evaluation. Continuous Integration (CI) boundaries process agent-opened PRs identically to human-opened PRs, executing deterministic evaluations alongside unit tests.

### Stage 6: The Operating System

Team composition transforms. Sprints include computational agent slots alongside human engineers. TDD becomes a strict architectural requirement to prevent agents from optimizing against weak constraints. Shared context (project memory, MCP servers) is managed as centralized infrastructure.

### Stage 7: The Smart Factory

Agents assemble and deliver complete, cohesive units of work with minimal human interaction. However, execution remains localized. Agents are invoked from individual developer environments, running local loops and opening PRs from localized sandboxes without a shared execution registry.

### Stage 8: The Autonomous Factory

Agents are fully decoupled from local hardware and deployed onto shared infrastructure. Executions are scheduled within isolated sandboxes featuring centralized tracing and logging. The system treats cyclic operations (dependency updates, security patches) as autonomous daemon tasks subject to programmatic success criteria. Organizational standards are encoded directly into automated evaluation suites rather than human knowledge.

## Data Structures

To deterministically evaluate compliance with this framework, systems MUST implement the following strict JSON schema for maturity assessments.

### MaturityAssessment Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MaturityAssessment",
  "type": "object",
  "properties": {
    "organization_id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique identifier of the evaluated entity."
    },
    "current_stage": {
      "type": "integer",
      "minimum": 1,
      "maximum": 8,
      "description": "The current verified operational stage."
    },
    "stage_configuration": {
      "type": "object",
      "properties": {
        "governance_enabled": {
          "type": "boolean",
          "description": "True if enterprise governance and SCIM/SSO are enforced."
        },
        "context_sharing_level": {
          "type": "string",
          "enum": ["NONE", "INDIVIDUAL", "TEAM", "ENTERPRISE"],
          "description": "The scope of context and skill sharing."
        },
        "infrastructure_hosting": {
          "type": "string",
          "enum": ["LOCAL", "SHARED_INFRASTRUCTURE"],
          "description": "The execution environment of the autonomous agents."
        }
      },
      "required": ["governance_enabled", "context_sharing_level", "infrastructure_hosting"],
      "additionalProperties": false
    }
  },
  "required": ["organization_id", "current_stage", "stage_configuration"],
  "additionalProperties": false
}
```

## Backward Compatibility

This standard operates at the organizational and process level, and is fully backward compatible with the deterministic control layer and cognitive isolation boundaries defined in `SDIC-RFC-0001` and the core SDIC-1 draft. No existing schemas or runtime environments require modification to implement this framework.
