---
title: "SDIC-ERC-0010 : Filtrage Standardisé des Injections de Prompt à l'Exécution"
author: "Jules <jules@example.com>"
status: Draft
type: Standards Track
created: 2026-08-01
---

[English](SDIC-ERC-0010.md) | [Français](SDIC-ERC-0010.fr.md)

## Résumé

Cette norme propose un modèle d'architecture pour atténuer les attaques par injection de prompt au niveau de l'exécution au sein du cadre SDIC-1. Elle vise à améliorer le Pilier I (Isolation Cognitive) en introduisant un pare-feu sémantique déterministe qui assainit le contexte injecté avant qu'il n'atteigne le bac à sable de l'IA (Sandbox).

## 1. Introduction

Pour comprendre le Pare-Feu Sémantique à l'Exécution (Runtime Semantic Firewall), imaginez une usine municipale de filtration d'eau ultramoderne. Le réservoir de la ville collecte de l'eau de diverses sources, y compris des précipitations naturelles imprévisibles et des ruissellements potentiellement contaminés.

Si cette eau brute et non filtrée était pompée directement dans les maisons immaculées des citoyens, l'ensemble du système d'eau pourrait être corrompu, entraînant des maladies généralisées. Par conséquent, avant que l'eau ne touche le réseau de distribution principal de la ville, elle doit passer par une membrane de filtration physique et stricte. Cette membrane n'est pas intelligente ; elle est purement mécanique. Elle permet aux molécules d'eau de passer tout en bloquant physiquement toute matière particulaire supérieure à une taille spécifique et rigoureusement définie.

Dans l'architecture SDIC-1 :

- **L'Eau Brute :** Cela représente les entrées fournies par l'utilisateur ou les réponses d'API externes qui sont potentiellement chargées de tentatives malveillantes d'injection de prompt.
- **Les Maisons des Citoyens :** Il s'agit du Bac à Sable de l'IA (Le Modèle Cognitif). Si des instructions malveillantes atteignent le modèle sans entrave, il pourrait contourner ses contraintes initiales.
- **La Membrane de Filtration Physique :** C'est le Pare-Feu Sémantique à l'Exécution. Il n'essaie pas de "comprendre" le texte. Au lieu de cela, il applique des règles structurelles et déterministes strictes (comme la suppression de délimiteurs non autorisés ou la validation des types d'entrée) pour éliminer les charges utiles malveillantes avant qu'elles ne puissent interagir avec le prompt système.

## 2. Justification & Impact Architectural

À mesure que les modèles deviennent plus performants, la surface d'attaque pour les injections de prompt via des entrées utilisateur manipulées augmente. S'appuyer uniquement sur l'alignement interne du modèle ou sur la validation finale du schéma JSON (Pilier II) est insuffisant ; si un modèle est complètement détourné, il pourrait générer des sorties adversariales conçues pour exploiter des vulnérabilités dans la couche d'analyse elle-même.

Le Pare-Feu Sémantique à l'Exécution introduit une couche de sécurité préventive essentielle. En assainissant de manière déterministe le contexte *avant* qu'il ne soit injecté dans le prompt de l'IA, nous réduisons considérablement la probabilité d'un détournement cognitif réussi.

## 3. Changements Proposés pour la Spécification

### 3.1. Composant Cible

Cette mise à niveau cible spécifiquement le **Pilier I : Isolation Cognitive**, établissant une étape de prétraitement obligatoire avant que le contexte filtré n'atteigne le Bac à Sable de l'IA.

### 3.2. Règle d'Assainissement du Contexte

Avant que toute donnée externe (entrée utilisateur, charge utile d'API externe) ne soit concaténée avec le prompt système, elle doit être validée par rapport à un schéma strict pour s'assurer qu'elle ne contient aucun code exécutable ou délimiteur de remplacement d'instruction.

### 3.3. Schéma d'Entrée Standardisé

L'application hôte DOIT appliquer le Schéma JSON suivant sur toutes les entrées non fiables destinées au contexte cognitif.

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

*Remarque : Le modèle regex `^[^<>{}\\[\\]]*$` interdit explicitement le formatage courant et les caractères structurels utilisés dans l'injection de prompt (balises XML, crochets JSON) dans la charge utile brute.*

### 3.4. Diagramme de Séquence

```text
+-------------+                 +-----------------------------+               +----------------+
| Source Non  |                 | Pare-Feu Sémantique         |               | IA / LLM       |
| Fiable      |                 | (Application Hôte)          |               | Sandbox        |
+-------------+                 +-----------------------------+               +----------------+
      |                                       |                                       |
      | 1. Soumettre Données                  |                                       |
      |-------------------------------------->|                                       |
      |                                       | 2. Validation Schéma & Regex          |
      |                                       |-----------------------------          |
      |                                       |                            |          |
      |                                       |<----------------------------          |
      |                                       |                                       |
      |                                       | 3. Rejet si Invalide                  |
      |<--------------------------------------| (Abandonner Requête)                  |
      |                                       |                                       |
      |                                       | 4. Injecter Contexte Assaini          |
      |                                       |-------------------------------------->|
      |                                       |                                       |
```

## 4. Rétrocompatibilité

Cette spécification est entièrement rétrocompatible. Elle agit comme une étape de prétraitement optionnelle (mais fortement recommandée) pour l'application hôte et ne modifie pas le registre d'actions principal (Action Ledger) ni les couches finales de déterminisme sémantique définies dans la version 1.0.0-draft.
