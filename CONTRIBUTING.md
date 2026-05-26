[English](CONTRIBUTING.md) | [Français](CONTRIBUTING.fr.md)

# Contributing to SDIC-1

We welcome contributions to the SDIC-1 (Systemic Deterministic Integration of AI Capabilities) standard. This repository consists entirely of markdown specifications. To ensure the highest quality of the standard, we enforce strict contribution guidelines.

## General Guidelines

- **Bilingual Documentation:** The project's documentation must be bilingual, providing both French and English versions with cross-navigation links at the top of each file. If you add or update a file, you must provide its translated counterpart.
- **No Corporate Fluff:** All Markdown files must be perfectly formatted, syntactically correct, and free of corporate fluff or placeholder text.
- **Feynman-style Analogies:** When writing or expanding standard documentation, include Feynman-style analogies to provide deep technical clarity and elegance.
- **Concrete Examples:** Concrete examples, applications, and algorithms are highly welcomed alongside explanations to maintain the technical depth of the standard.

## Technical Specifications

- **Language-Agnostic:** Technical specifications must be language-agnostic and highly technical.
- **Deterministic Validations:** Use strict JSON Schema examples or UML ASCII sequence diagrams where applicable.
- **Strict JSON Schema:** When defining JSON Schemas within the repository, enforce strictness by setting `"additionalProperties": false` at the top level (and on applicable nested objects) to prevent unspecified keys.

## Request for Comments (RFCs)

- **Standard Internet Protocol Format:** RFC proposals must follow the classic Internet Protocol (IETF RFC) format.
- **Directory:** RFCs must be stored in the `ecosystem/RFCs/` directory.

## Commit Guidelines

- **Conventional Commits:** Commits must strictly use the [Conventional Commits](https://www.conventionalcommits.org/) format.

## Pull Request Guidelines

Pull Request descriptions must strictly adhere to the following layout containing these exact headers:

1. **Summary of Changes**
2. **Rationale & Architectural Impact**
3. **Proposed Specification Changes**
4. **Backward Compatibility**

## Formatting

- Markdown formatting is verified and fixed using `markdownlint-cli`.
- We use a relaxed `.markdownlint.json` configuration (disabling rules like MD013 for line length and MD041 for first-line headings). Ensure your changes comply with the configuration before submitting.
