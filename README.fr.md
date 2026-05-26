[English](README.md) | [Français](README.fr.md)

# SDIC-1: Systemic Deterministic Integration of AI Capabilities

Author: Charles EDOU NZE
Status: Draft
Type: Standards Track
Created: 2026-05-25
---

## Abstract

Ce standard définit une architecture logicielle stricte permettant l'intégration de capacités cognitives probabilistes (telles que les Grands Modèles de Langage ou LLM) au sein d'applications d'entreprise déterministes. L'objectif premier de SDIC-1 est de supprimer l'accès direct des agents autonomes aux couches de persistance et d'exécution, en introduisant une barrière d'isolation et un registre d'intentions standardisé.

## 1. Motivation

L'essor des architectures dites "AI-Native" et du "Vibe Coding" pose un risque systémique pour les industries hautement régulées (Banque, Industrie, Santé). Un modèle probabiliste ne garantit pas la répétabilité de sa structure de sortie, ce qui le rend incompatible avec les exigences de sécurité et de conformité logicielle traditionnelles.

À l'image des standards de la programmation structurée ou des protocoles décentralisés (EIP/ERC), SDIC-1 formalise les frontières de sécurité indispensables pour transformer une force générative imprévisible en un composant applicatif maîtrisable et auditable.

## 2. Specification

Le protocole repose sur l'implémentation obligatoire de quatre couches d'abstractions logicielles indépendantes du langage de programmation utilisé (Agnostic Stack).

```text
+-------------------------------------------------------------+
|                      Application Hôte                       |
|   (Système Déterministe : Symfony, React, Go, etc.)         |
+------------------------------+------------------------------+
                               |
                   [1. Contexte Filtré & Prompt]
                               |
                               v
+-------------------------------------------------------------+
|                     IA / LLM Sandbox                        |
|   (Isolation Cognitive - Analyse & Émission d'Intention)    |
+------------------------------+------------------------------+
                               |
                    [2. Intention JSON brute]
                               |
                               v
+-------------------------------------------------------------+
|                Couche de Contrôle Déterministe              |
|   (Validation de Schéma Strict / Guardrails)               |
+------------------------------+------------------------------+
                               |
                  [3. Intention Validée / Rejet]
                               |
                               v
+-------------------------------------------------------------+
|                        Action Ledger                        |
|   (Registre Immuable de Persistance et d'Exécution)        |
+-------------------------------------------------------------+
```

### 2.1. Isolation Cognitive (The Sandbox Boundary)

L'environnement d'exécution de l'IA (LLM, Agent) doit être structurellement isolé de l'infrastructure hôte.

- **Règle d'or :** L'IA ne possède aucun droit d'écriture réseau, aucun accès direct à la base de données principale, et aucun droit d'appel d'API mutatives.
- **Contrat d'interface :** L'IA agit exclusivement en tant que transformateur sémantique. Elle reçoit un flux textuel filtré et retourne uniquement un objet structuré représentant une intention d'action, sans pouvoir l'exécuter elle-même.

### 2.2. Déterminisme Sémantique (Zero-Trust Schema Validation)

Toute donnée sortant de la Sandbox Cognitive doit être traitée comme une saisie utilisateur non sécurisée (Untrusted Input).

- **Mécanisme de contrôle :** L'application hôte doit intercepter le payload de l'IA et le soumettre à une validation de schéma strict (ex: JSON Schema).
- **Gestion des déviations :** Si la structure de la réponse de l'IA diverge du schéma attendu (changement de clé, type de donnée erroné, omission d'un champ obligatoire), l'application hôte doit lever une exception logicielle immédiate. Le système doit alors basculer sur un processus de secours déterministe (Fallback) ou requérir une arbitrage humain.

### 2.3. L'Action Ledger (L'Auditabilité Temporelle)

Chaque action validée issue d'une intention de l'IA doit être enregistrée de manière immuable avant son exécution.
Le registre d'action (Action Ledger) doit sauvegarder un instantané contenant les structures de données suivantes :

| Champ | Type | Description |
|---|---|---|
| transaction_id | UUIDv4 | Identifiant unique et immuable de la transaction. |
| timestamp | DateTime | Horodatage précis de l'interaction à l'échelle du système. |
| model_snapshot | String | Version exacte du modèle utilisé (ex: claude-3-5-sonnet, gpt-5-preview). |
| hyperparameters | Object | Configuration du modèle au runtime (Température, Top_P, Max Tokens). |
| cognitive_context | Text | Copie exacte du prompt système et des données injectées via RAG. |
| raw_response | Text | Réponse brute renvoyée par le modèle avant validation. |

### 2.4. L'Hydratation Chirurgicale (Context Efficiency Boundary)

Afin de minimiser la surface d'attaque par injection de prompt, la latence et les coûts opérationnels, la fenêtre de contexte envoyée à l'IA doit être restreinte au strict minimum nécessaire.

- **Principe d'atténuation :** Il est interdit de transmettre des tables de bases de données entières ou des historiques complets non filtrés.
- **Spécification :** L'application hôte doit utiliser des stratégies d'indexation vectorielle (RAG) ou de filtrage applicatif pour extraire uniquement l'atome de donnée requis pour la résolution de la tâche en cours.

## 3. Rationale

Le choix d'une architecture découpée et asynchrone plutôt que d'une intégration native des agents d'exécution (comme le Function Calling direct ou les agents CLI autonomes) est motivé par la nécessité de garantir la gouvernance des systèmes.
En déléguant la génération de l'intention à l'IA et la validation/exécution au code traditionnel, SDIC-1 permet aux ingénieurs seniors de conserver le contrôle absolu sur les règles métiers de l'entreprise.

## 4. Security Considerations

- **Prompt Injection Mitigation :** Le respect strict du Pilier II (Déterminisme Sémantique) garantit que même si un attaquant parvient à manipuler le modèle via une injection de prompt, l'output généré sera rejeté par le validateur de schéma si l'intention ne correspond pas aux structures autorisées.
- **Data Leakage Prevention :** Le respect du Pilier IV limite l'exposition accidentelle de données sensibles de l'entreprise vers des modèles tiers ou mutualisés.
