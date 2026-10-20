---
title: "SDIC-ERC-0020: Standardisation de la Structure de Données du Registre d'Actions pour la Synchronisation Multi-Agents"
author: "Jules <jules@example.com>"
status: Draft
type: Standards Track
created: 2026-10-20
---

[English](SDIC-ERC-0020.md) | [Français](SDIC-ERC-0020.fr.md)

## Résumé

Cette norme propose un modèle architectural et un schéma JSON strict pour le composant "Registre d'Actions" (Action Ledger) du framework SDIC-1 afin de faciliter une synchronisation multi-agents sécurisée et déterministe (Consensus d'Essaim). Elle confine mathématiquement la communication inter-agents, garantissant que de multiples LLMs probabilistes peuvent orchestrer des tâches asynchrones sans muter directement un état partagé ni introduire de conditions de course.

## 1. Introduction

Imaginez une tour de contrôle aérien coordonnant une flotte d'avions autonomes. Si les avions tentent de communiquer et de coordonner leurs trajectoires de vol directement entre eux sur une fréquence radio ouverte, le résultat est le chaos. Une erreur de transmission, un signal mal compris ou une interférence malveillante pourrait amener deux avions à tenter d'occuper le même espace aérien simultanément.

Pour éviter cela, nous introduisons le manifeste de vol centralisé et le répartiteur. Il est strictement interdit aux avions (les agents autonomes) de se diriger les uns les autres. Au lieu de cela, lorsqu'un avion souhaite modifier son altitude, il envoie une demande hautement structurée et signée cryptographiquement (l'Intention) au répartiteur (la Couche de Contrôle Déterministe). Le répartiteur vérifie la demande par rapport à des règles aéronautiques strictes. Si elle est valide, le répartiteur met à jour le manifeste de vol central (le Registre d'Actions). Tous les autres avions lisent en permanence ce manifeste immuable pour comprendre l'état actuel de l'espace aérien et ajuster leur comportement en conséquence.

Dans l'architecture SDIC-1 :

- **Les avions autonomes** représentent les agents d'IA probabilistes. Ils sont intelligents mais ne peuvent pas modifier directement l'état partagé du système.
- **Le répartiteur** représente la Couche de Contrôle Déterministe, appliquant une validation stricte du schéma JSON.
- **Le manifeste de vol central** est le Registre d'Actions Standardisé. Il sert de source de vérité unique et immuable qui synchronise l'essaim de manière déterministe.

## 2. Justification et Impact Architectural

À mesure que les architectures multi-LLMs évoluent, les agents doivent collaborer de manière transparente tout en respectant l'isolation cognitive imposée par le SDIC-1 (Pilier I). Les implémentations actuelles manquent d'un support standardisé pour que les agents diffusent et valident leurs intentions en toute sécurité au sein d'un Consensus d'Essaim. La communication non structurée d'agent à agent présente des risques importants :

- **Dérive d'état :** Sans une vérité centralisée, les agents peuvent se désynchroniser, agissant sur un contexte obsolète ou conflictuel.
- **Propagation d'Injection de Prompt :** La communication directe permet à un agent piraté de propager une charge utile de prompt malveillante au reste de l'essaim.
- **Conditions de course :** Des actions simultanées sans mécanisme de verrouillage déterministe conduisent à des mutations d'état imprévisibles.

Un schéma standardisé de Registre d'Actions garantit une coordination d'agents immuable, déterministe et vérifiée par cryptographie. En imposant `"additionalProperties": false`, le schéma dissipe instantanément les intentions mal formées et neutralise les charges utiles d'injection de prompt tentant d'ajouter des données non validées.

## 3. Modifications Proposées de la Spécification

### 3.1. Composant Cible

Cette spécification cible le **Pilier IV : Le Registre d'Actions**, en définissant une structure déterministe pour les propositions d'intentions multi-agents et le consensus.

### 3.2. Schéma JSON du Registre d'Actions Multi-Agents

Les agents DOIVENT formater leurs propositions de synchronisation exactement selon le Schéma JSON suivant. Tous les objets DOIVENT imposer `"additionalProperties": false`. Les signatures cryptographiques DOIVENT valider une représentation canonique de l'entrée complète (à l'exclusion du champ de signature lui-même) pour empêcher les attaques par rejeu et par changement d'étiquette.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentActionLedgerEntry",
  "type": "object",
  "properties": {
    "transaction_id": {
      "type": "string",
      "format": "uuid"
    },
    "correlation_id": {
      "type": "string",
      "format": "uuid",
      "description": "Utilisé pour lier de multiples intentions dans une saga distribuée"
    },
    "agent_id": {
      "type": "string",
      "format": "uuid"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "proposed_action": {
      "type": "string",
      "enum": ["propose_state_mutation", "acknowledge_state", "request_consensus"]
    },
    "payload": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "target_resource": { "type": "string" },
        "mutation_type": { "type": "string", "enum": ["create", "update", "delete"] },
        "mutation_data": { "type": "string", "description": "Représentation sous forme de chaîne déterministe de la modification" }
      },
      "required": ["target_resource", "mutation_type", "mutation_data"]
    },
    "proof_of_intent": {
      "type": "string",
      "description": "Signature cryptographique de l'intention canonique"
    }
  },
  "required": [
    "transaction_id",
    "correlation_id",
    "agent_id",
    "timestamp",
    "proposed_action",
    "payload",
    "proof_of_intent"
  ],
  "additionalProperties": false
}
```

### 3.3. Diagramme de Séquence du Consensus d'Essaim

Le diagramme de séquence UML ASCII suivant illustre la propagation déterministe des modifications d'état via le Registre d'Actions, empêchant la mutation directe d'agent à agent.

```text
+---------+               +-----------------------+              +-----------------+               +---------+
| Agent A |               | Contrôle Déterministe |              | Registre Action |               | Agent B |
+---------+               +-----------------------+              +-----------------+               +---------+
     |                                |                                   |                             |
     | 1. Émet l'Intention (JSON)     |                                   |                             |
     |------------------------------->|                                   |                             |
     |                                | 2. Validation Schéma              |                             |
     |                                |    (additionalProperties: false)  |                             |
     |                                |-----------------------------      |                             |
     |                                |                            |      |                             |
     |                                |<----------------------------      |                             |
     |                                |                                   |                             |
     |                                | 3. Vérification Crypto            |                             |
     |                                |    (Charge Utile Canonique)       |                             |
     |                                |-----------------------------      |                             |
     |                                |                            |      |                             |
     |                                |<----------------------------      |                             |
     |                                |                                   |                             |
     |                                | 4. Valider dans le Registre       |                             |
     |                                |---------------------------------->|                             |
     |                                |                                   | 5. Réhydratation de l'État  |
     |                                |                                   |---------------------------->|
     |                                |                                   |                             |
```

## 4. Rétrocompatibilité

Cette spécification est entièrement rétrocompatible. Les systèmes implémentant la `v1.0.0-draft` du SDIC-1 peuvent supporter le consensus multi-agents en introduisant le `correlation_id` et en standardisant la structure de charge utile du registre sans nécessiter de modifications de la Sandbox IA sous-jacente ou des règles de filtrage déterministes existantes. Les systèmes à agent unique peuvent ignorer en toute sécurité les champs spécifiques multi-agents.

## Droit d'auteur

Les droits d'auteur et les droits connexes sont renoncés via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
