---
title: "SDIC-RFC-0004: Standardisation du Registre d'Actions pour l'Orchestration Multi-Agents Asynchrone"
author: "Jules <jules@example.com>"
status: Draft
type: Standards Track
created: 2026-06-01
---

## Résumé

Cette RFC standardise la structure de données du "Registre d'Actions" (Action Ledger) pour l'orchestration multi-agents asynchrone au sein du framework SDIC-1. Afin de prévenir la corruption d'état lorsque plusieurs agents autonomes opèrent simultanément, cette spécification introduit un schéma déterministe et sécurisé par cryptographie pour l'enregistrement des intentions et l'obtention d'un consensus d'essaim (swarm consensus) avant la transition d'état.

## 1. Introduction

À mesure que les systèmes multi-agents évoluent, la probabilité de collision d'états et d'effets en cascade imprévisibles augmente de manière exponentielle. Lorsque de multiples agents probabilistes agissent sur un état partagé, les bases de données monolithiques traditionnelles ne parviennent pas à fournir l'isolation cognitive requise par le SDIC-1.

Imaginez un système de Contrôle de la Circulation Aérienne (ATC) gérant un essaim de drones de livraison autonomes. Les drones eux-mêmes (les agents probabilistes) ne décident jamais d'atterrir simultanément sur la même piste. Au lieu de cela, chaque drone calcule sa trajectoire optimale et envoie une Demande d'Atterrissage formelle (l'intention) à la tour de contrôle. La tour de contrôle (l'hôte déterministe) agit comme un journal immuable et un séquenceur. Elle vérifie chaque demande, s'assure qu'aucun drone n'essaie d'utiliser la piste en même temps selon des règles physiques rigides, puis autorise l'atterrissage un par un. Les drones ne se coordonnent pas directement entre eux en modifiant leurs trajectoires arbitrairement ; ils se synchronisent via le registre central et déterministe de la tour de contrôle.

En transposant cette analogie à notre architecture technique :

- **Les Drones :** Ce sont les agents LLM autonomes opérant dans leurs bacs à sable (sandboxes) isolés. Ils calculent des solutions optimales basées sur un raisonnement probabiliste mais n'ont pas l'autorité directe d'exécuter des changements d'état.
- **La Demande d'Atterrissage :** Il s'agit de l'`Intent_Proposal` (Proposition d'Intention) strictement défini et conforme à un schéma JSON. Cela représente la volonté sémantique de l'IA de modifier l'état.
- **La Tour de Contrôle :** Cela représente l'Application Hôte. Elle reçoit les demandes, les valide selon des règles déterministes et les séquence.
- **Le Journal Immuable :** C'est le Registre d'Actions. Chaque `Intent_Proposal` validé est enregistré ici cryptographiquement avant qu'aucun changement d'état réel ne se produise.
- **L'Autorisation d'Atterrir :** Il s'agit du Consensus d'Essaim et de la Transition d'État subséquente. L'hôte garantit que seules des intentions non conflictuelles et mathématiquement vérifiées sont exécutées.

## 2. Justification et Impact Architectural

La vulnérabilité principale dans les flux de travail multi-LLM est le manque d'arbitrage synchronisé et déterministe entre les intentions conflictuelles des agents. Si deux agents tentent de muter une ressource simultanément sans structure stricte, le système entre dans un état non déterministe.

En imposant un Registre d'Actions standardisé et cryptographique, nous obtenons :

1. **Déterminisme Temporel :** Toutes les intentions des agents sont strictement sérialisées.
2. **Non-Répudiation Cryptographique :** Les signatures garantissent que les intentions ne peuvent être altérées ou usurpées après émission.
3. **Fondation du Consensus d'Essaim :** Fournit la structure de données exacte requise pour que les agents lisent les intentions passées et formulent des propositions subséquentes cohérentes sans bavardage direct entre agents.

## 3. Spécification

### 3.1. Schéma Standardisé de l'Entrée du Registre d'Actions

Le schéma JSON suivant définit la structure requise pour toute entrée validée dans le Registre d'Actions dans un contexte multi-agents. L'adhésion stricte est imposée (`"additionalProperties": false`).

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ActionLedgerEntry",
  "type": "object",
  "properties": {
    "transaction_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique et immuable de la transaction."
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "Marqueur temporel précis au niveau du système."
    },
    "agent_id": {
      "type": "string",
      "description": "Identifiant de l'agent autonome émettant l'intention."
    },
    "model_snapshot": {
      "type": "string",
      "description": "Version exacte du modèle utilisé."
    },
    "intent_type": {
      "type": "string",
      "enum": ["READ", "MUTATE", "PROPOSE", "VOTE"]
    },
    "payload": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "target_resource": { "type": "string" },
        "action": { "type": "string" },
        "parameters": { "type": "object", "additionalProperties": true }
      },
      "required": ["target_resource", "action"]
    },
    "proof_of_intent": {
      "type": "string",
      "description": "Signature cryptographique de l'intention canonique."
    }
  },
  "required": [
    "transaction_id",
    "timestamp",
    "agent_id",
    "model_snapshot",
    "intent_type",
    "payload",
    "proof_of_intent"
  ],
  "additionalProperties": false
}
```

### 3.2. Flux d'Orchestration Multi-Agents Asynchrone

L'interaction entre les agents et l'hôte doit suivre un schéma asynchrone strict, entièrement médié par le Registre d'Actions.

```text
+---------+          +-------------------+          +-------------+
| Agent A |          | Action Ledger (AL)|          | Host System |
+---------+          +-------------------+          +-------------+
     |                         |                           |
     | 1. Read Current State   |                           |
     |------------------------>|                           |
     |                         |                           |
     | 2. Compute Intent       |                           |
     | (Probabilistic)         |                           |
     |                         |                           |
     | 3. Submit Intent        |                           |
     | (Signed JSON)           |                           |
     |------------------------>|                           |
     |                         | 4. Validate Schema & Sig. |
     |                         |-------------------------->|
     |                         |                           |
     |                         |                           | 5. If Valid, Sequence
     |                         |                           |    and Attempt Execution
     |                         |                           |
     |                         | 6. Record Outcome         |
     |                         |<--------------------------|
     |                         |                           |
     | 7. Read Updated State   |                           |
     |<------------------------|                           |
     |                         |                           |
```

## 4. Considérations de Sécurité

- **Canonicalisation :** La signature `proof_of_intent` DOIT valider une version canonisée de l'objet JSON (ex : clés triées, pas d'espaces) moins le champ `proof_of_intent` lui-même.
- **Attaques par Rejeu :** Les champs `transaction_id` et `timestamp` doivent être strictement validés par l'hôte pour garantir qu'une intention n'est pas exécutée plus d'une fois.

## 5. Compatibilité Ascendante

Cette RFC étend le concept initial du Registre d'Actions décrit dans le brouillon SDIC-1 v1.0.0. Il introduit des structures plus rigides pour les capacités multi-agents mais est entièrement rétrocompatible avec les systèmes mono-agent qui implémentent la philosophie de base de SDIC-1.
