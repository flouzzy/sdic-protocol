---
rfc: 9
title: Génération Guidée par Machine à États Finis pour la Validation de Garde-Fous à Latence Zéro
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
created: 2026-07-15
---

[English](SDIC-RFC-0009.md) | [Français](SDIC-RFC-0009.fr.md)

## Résumé

Ce standard propose une optimisation architecturale du protocole SDIC-1 afin de réduire drastiquement la surcharge de latence de la couche de Garde-fous (Guardrails). En précompilant le Schéma JSON strict en une Machine à États Finis (FSM) déterministe, la couche de Garde-fous peut valider le flux d'intentions sémantiques jeton par jeton (token-by-token). Cela garantit que les générations non conformes sont interrompues instantanément, réduisant le gaspillage de calcul et éliminant complètement la surcharge de latence associée à la validation de schéma post-génération.

## Motivation

Au sein de l'architecture SDIC-1, la couche de Déterminisme Sémantique impose que toutes les intentions générées par l'IA adhèrent strictement à un Schéma JSON prédéfini. Traditionnellement, cela nécessite que le Grand Modèle de Langage (LLM) termine toute sa séquence de génération, après quoi l'application hôte analyse la charge utile JSON complète et la valide. Dans des environnements d'entreprise exigeant une orchestration agentique à haute fréquence, ce modèle d'attente et de validation introduit des surcharges de latence inacceptables et gaspille des ressources de calcul sur la génération de jetons qui seront finalement invalides. Un mécanisme est nécessaire pour faire appliquer l'adhérence au schéma avec un ajout de latence nul tout en maintenant une isolation déterministe absolue.

## L'Analogie de l'Aiguilleur de Train à Grande Vitesse

Imaginez un réseau de trains à grande vitesse où un répartiteur envoie des trains vers diverses destinations. Au lieu de laisser un train terminer tout son voyage pour vérifier s'il est arrivé à la bonne destination en toute sécurité et le renvoyer si ce n'est pas le cas, le réseau utilise des aiguilleurs de voie automatisés à chaque jonction. Les aiguilleurs connaissent les itinéraires valides exacts à l'avance. À mesure que le train se déplace, chaque aiguilleur vérifie instantanément le segment suivant. Si le train tente de prendre une voie invalide, l'aiguilleur arrête immédiatement le train en toute sécurité avant qu'il ne puisse provoquer une collision, plutôt que d'attendre la fin du voyage.

Dans cette architecture :

- **Le train à grande vitesse** représente les jetons en flux continu de l'intention JSON générée par le modèle d'IA.
- **Le voyage complet** représente la génération complète de la charge utile de l'intention sémantique.
- **Les aiguilleurs de voie automatisés** représentent la Machine à États Finis (FSM) déterministe dérivée du Schéma JSON au niveau de la couche de Garde-fous.
- **Le déraillement en toute sécurité aux voies invalides** représente le rejet instantané du jeton et la résiliation de la connexion par la couche de Garde-fous, empêchant la surcharge de latence de la génération complète et de la post-validation consécutive.

## Spécification

### 1. Compilation de la Machine à États Finis

Avant exécution, la couche de Garde-fous DOIT compiler le Schéma JSON attendu en une Machine à États Finis (FSM) déterministe. La FSM représente toutes les séquences valides de caractères (ou de jetons) qui satisfont le schéma strict, y compris les contraintes `"additionalProperties": false`.

### 2. Validation de Flux Jeton par Jeton

Pendant la phase de génération de l'IA, la sortie DOIT être transmise en continu à la couche de Garde-fous. Pour chaque jeton émis $T_i$, la couche de Garde-fous met à jour l'état de la FSM.

Soit $S$ l'état de la FSM.
$S_{t+1} = \delta(S_t, T_i)$

Si $\delta(S_t, T_i)$ effectue une transition vers un état invalide (c'est-à-dire que le jeton viole le schéma), la couche de Garde-fous DOIT terminer instantanément le flux de génération et lever une exception de validation, interrompant ainsi prématurément l'exécution du modèle probabiliste de manière effective.

### 3. Schéma de Configuration de la FSM

Pour standardiser l'instanciation du validateur FSM, la configuration DOIT adhérer au Schéma JSON strict suivant :

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "FSMGuardrailConfig",
  "type": "object",
  "properties": {
    "schema_id": {
      "type": "string",
      "format": "uuid"
    },
    "strict_mode": {
      "type": "boolean",
      "description": "Enforces compilation with additionalProperties: false universally."
    },
    "max_token_buffer": {
      "type": "integer",
      "minimum": 1
    },
    "abort_on_violation": {
      "type": "boolean"
    }
  },
  "required": [
    "schema_id",
    "strict_mode",
    "max_token_buffer",
    "abort_on_violation"
  ],
  "additionalProperties": false
}
```

### 4. Diagramme de Séquence

```text
+----------+                     +------------------+                  +------------------+
| AI Model |                     | Guardrails Layer |                  | Host Application |
+----------+                     +------------------+                  +------------------+
     |                                     |                                     |
     | 1. Stream Token T_1                 |                                     |
     |------------------------------------>|                                     |
     |                                     | 2. FSM State Transition             |
     | 3. Stream Token T_2                 |                                     |
     |------------------------------------>|                                     |
     |                                     | 4. FSM Invalid State Reached        |
     | 5. Terminate Stream Connection      |                                     |
     |<------------------------------------|                                     |
     |                                     | 6. Validation Exception             |
     |                                     |------------------------------------>|
```

## Compatibilité Ascendante

Cette optimisation est entièrement rétrocompatible avec le v1.0.0-draft de SDIC-1. Elle ne modifie pas le format de sortie final mais optimise le pipeline de validation. Les systèmes incapables de validation en continu peuvent basculer de manière transparente sur une validation de schéma de charge utile complète post-génération sans aucune modification structurelle.
