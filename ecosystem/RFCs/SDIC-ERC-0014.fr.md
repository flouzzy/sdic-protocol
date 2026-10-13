---
title: "SDIC-ERC-0014: Standardized Low-Latency Streaming Guardrails"
author: "Jules <jules@example.com>"
status: Draft
type: Standards Track
created: 2026-10-01
---

[English](SDIC-ERC-0014.md) | [Français](SDIC-ERC-0014.fr.md)

## Abstract

Ce standard introduit une optimisation architecturale pour le protocole SDIC-1 visant à réduire la latence de la couche de garde-fous (Pilier II : Déterminisme Sémantique). En imposant une validation déterministe du flux de tokens (token-streaming), les applications hôtes peuvent évaluer des intentions JSON partielles en temps réel, interrompant instantanément la génération en cas de violation de schéma sans attendre la fin de la réponse probabiliste.

## 1. Introduction

Imaginez un train de marchandises à grande vitesse s'approchant d'une frontière internationale strictement réglementée. Traditionnellement, le contrôle aux frontières oblige le train entier à s'arrêter complètement dans une immense cour d'inspection. Ce n'est qu'une fois le tout dernier wagon entré, et le manifeste complet vérifié par rapport à la cargaison, que le train est autorisé à poursuivre sa route. Si un article interdit est trouvé dans le dernier wagon, le train entier est rejeté après un retard important. Cette approche, bien que sûre, crée une congestion et une latence énormes.

Imaginez maintenant un point de contrôle d'inspection continue et moderne. Alors que le train roule à grande vitesse, des capteurs spécialisés scannent chaque wagon individuellement en temps réel. Au moment où un wagon passe, son contenu est vérifié par rapport au manifeste pour cette section spécifique. Si une violation est détectée dans le troisième wagon, la voie de triage dévie immédiatement le train vers une voie de garage sécurisée, arrêtant le processus bien avant que le reste du train n'arrive. Si aucune violation n'est trouvée, le train ne perd jamais son élan.

Dans l'architecture SDIC-1 :

- **Le train de marchandises à grande vitesse** est le flux de tokens généré par le modèle d'IA.
- **L'immense cour d'inspection** représente la validation de schéma JSON traditionnelle et bloquante qui attend que la réponse complète du LLM soit terminée avant l'analyse.
- **Le point de contrôle d'inspection continue et moderne** est la couche de Validation Déterministe en Continu. Elle évalue des structures JSON partielles en temps réel à mesure que les tokens sont émis.
- **La voie de triage déviant le train** représente la résiliation immédiate de la connexion et le rejet de l'intention à la milliseconde où une déviation structurelle ou une violation de schéma est détectée, réduisant ainsi drastiquement la latence et le gaspillage de calcul.

## 2. Rationale & Architectural Impact

Dans l'intégration traditionnelle de l'IA en entreprise, la couche de garde-fous agit comme une opération bloquante. L'IA génère l'intention brute complète, et ce n'est qu'alors que la couche de contrôle déterministe la valide par rapport au schéma JSON strict. Si le modèle hallucine une propriété invalide tôt dans la génération, le système attend toujours que la réponse entière se termine avant de la rejeter, causant une latence importante et gaspillant des ressources de calcul.

En implémentant la validation déterministe en continu, le protocole SDIC réduit considérablement la latence de bout en bout. Il protège les systèmes d'entreprise de l'imprévisibilité de l'IA en appliquant la frontière mathématique en continu. Si un modèle d'IA s'écarte du schéma strict, la génération est tuée instantanément, garantissant que les intentions non conformes ne consomment jamais de temps de traitement inutile.

## 3. Proposed Specification Changes

### 3.1. Target Component

Cette spécification cible le **Pilier II : Déterminisme Sémantique**, en optimisant le mécanisme de validation de schéma d'une opération bloquante post-génération vers un processus de validation continue en cours de vol.

### 3.2. Streaming Guardrail Configuration Schema

Pour standardiser le comportement de la couche de validation en continu, les applications hôtes DOIVENT configurer leurs garde-fous en utilisant le schéma JSON strict suivant. Il applique une stricte adhérence en définissant `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "StreamingGuardrailConfig",
  "type": "object",
  "properties": {
    "stream_id": {
      "type": "string",
      "format": "uuid"
    },
    "max_token_buffer": {
      "type": "integer",
      "minimum": 1,
      "maximum": 128
    },
    "chunk_timeout_ms": {
      "type": "integer",
      "minimum": 10,
      "maximum": 5000
    },
    "abort_on_first_violation": {
      "type": "boolean"
    }
  },
  "required": [
    "stream_id",
    "max_token_buffer",
    "chunk_timeout_ms",
    "abort_on_first_violation"
  ],
  "additionalProperties": false
}
```

### 3.3. Interaction Sequence Diagram

Le diagramme de séquence suivant illustre le processus d'inspection continue et de résiliation immédiate.

```text
+----------+                     +-----------------------+                    +-----------------+
| Modèle IA|                     | Contrôle en Continu   |                    | Action Ledger   |
+----------+                     +-----------------------+                    +-----------------+
     |                                      |                                          |
     | 1. Émet segment (Tokens 1-N)         |                                          |
     |------------------------------------->|                                          |
     |                                      | 2. Évaluation Schéma Partiel             |
     |                                      |-----------------------------             |
     |                                      |                            |             |
     |                                      |<----------------------------             |
     |                                      |                                          |
     | 3. Émet segment (Tokens N-M)         |                                          |
     |------------------------------------->|                                          |
     |                                      | 4. Violation Détectée !                  |
     |                                      |-----------------------------             |
     |                                      |                            |             |
     |                                      |<----------------------------             |
     |                                      |                                          |
     | 5. Tuer la Connexion (Avorter)       |                                          |
     |<-------------------------------------|                                          |
     |                                      | 6. Consigner l'Événement de Rejet        |
     |                                      |----------------------------------------->|
```

## 4. Backward Compatibility

Cette spécification est entièrement rétrocompatible. Elle fonctionne comme un mécanisme d'optimisation par-dessus la couche de Déterminisme Sémantique existante définie dans la v1.0.0-draft. Les applications hôtes qui ne prennent pas en charge le streaming peuvent se dégrader gracieusement vers la validation JSON standard et bloquante sans aucune perte des garanties de sécurité.

## Copyright

Copyright et droits connexes renoncés via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
