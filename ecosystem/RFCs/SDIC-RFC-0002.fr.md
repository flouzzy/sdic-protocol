---
eip: 2
title: Amélioration du Filtrage des Injections de Prompt au Niveau de l'Exécution
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
category: Core
created: 2026-06-02
---

[English](SDIC-RFC-0002.md) | [Français](SDIC-RFC-0002.fr.md)

## Résumé

Cette norme propose une extension architecturale au protocole SDIC-1 pour atténuer les attaques par injection de prompt de manière native au niveau de l'exécution (runtime). Elle spécifie une séquence de filtration déterministe et un schéma de configuration JSON strict pour la désinfection sémantique à l'exécution, garantissant que tout contexte malveillant injecté dans un modèle d'IA est mathématiquement isolé et supprimé avant exécution.

## Motivation

À mesure que les systèmes multi-agents autonomes traitent des entrées utilisateur non fiables au sein des piles d'entreprise, l'injection de prompt reste une vulnérabilité majeure. Bien que SDIC-1 impose des sorties rigoureuses basées sur un schéma JSON, des injections de prompt très sophistiquées pourraient tromper un LLM pour qu'il émette des schémas JSON valides contenant une logique malveillante intégrée dans le texte du payload lui-même. Pour encapsuler en toute sécurité la frontière d'isolation cognitive, un filtrage des injections de prompt au niveau de l'exécution est nécessaire.

## L'Analogie de la Station d'Épuration des Eaux

Imaginez une grande ville dépendant d'une rivière pour son approvisionnement en eau. La rivière représente le flux d'entrées et de contextes non structurés générés par les utilisateurs. Cette eau est fondamentalement chaotique et potentiellement contaminée. Nous ne pouvons pas simplement pomper cette eau brute directement dans le réseau d'eau potable de notre ville, ni supposer que l'eau est sûre simplement parce qu'elle a traversé un tuyau d'une forme spécifique.

Pour garantir la sécurité, nous devons établir une station d'épuration des eaux hautement réglementée. Lorsque l'eau brute pénètre dans la station, elle passe d'abord à travers de lourdes grilles pour retenir les gros débris. Ensuite, elle s'écoule à travers de fins filtres de sable et de charbon qui se lient aux impuretés microscopiques et aux métaux lourds. Enfin, l'eau est soumise à une lumière ultraviolette pour neutraliser toute menace biologique restante. Ce n'est qu'après avoir passé ces étapes déterministes et rigoureuses que l'eau propre est pompée dans les tuyaux standardisés de la ville.

Dans notre architecture SDIC-1 :

- **L'Eau Brute de la Rivière** représente les sorties de texte brutes et le contexte extrait générés par le grand modèle de langage (l'IA), qui peuvent contenir des injections de prompt cachées.
- **Les Grilles et les Filtres à Sable** représentent nos analyseurs déterministes d'expressions régulières et de listes de blocage, éliminant les modèles hostiles connus et les caractères d'échappement.
- **La Lumière Ultraviolette** agit comme l'analyseur d'intention sémantique, évaluant le payload isolé pour s'assurer qu'il ne tente pas de subvertir les limites déterministes.
- **L'Eau Propre Pompée dans des Tuyaux Standardisés** représente l'intention désinfectée qui passe avec succès la validation et correspond parfaitement au schéma JSON strictement défini, désormais sûre à exécuter dans le système déterministe.

## Spécification

### 1. Séquence de Filtration Déterministe à l'Exécution

Le processus de filtration intercepte la sortie de l'IA avant l'étape finale de validation du schéma JSON.

```text
+-----------------------+
| Sandbox de l'IA / LLM |
+-----------+-----------+
            |
            | 1. Intention JSON Brute
            v
+-----------------------+
|  Couche de Filtration |
|      à l'Exécution    |
+-----------+-----------+
            |
            | 2. Appliquer les Listes de Blocage et la Concordance de Modèles
            | 3. Vérification des Limites Sémantiques
            v
+-----------------------+
| Contrôle Déterministe |
|    (Schéma JSON)      |
+-----------------------+
```

### 2. Schéma de Configuration du Filtre d'Injection de Prompt

Chaque politique de filtrage DOIT adhérer au schéma JSON suivant, appliquant mathématiquement les limites du filtre d'injection à l'exécution. Les objets doivent imposer `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RuntimeFiltrationConfig",
  "type": "object",
  "properties": {
    "policy_id": {
      "type": "string",
      "format": "uuid"
    },
    "blocklists": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "allowed_patterns": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "max_payload_entropy": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "drop_on_failure": {
      "type": "boolean"
    }
  },
  "required": [
    "policy_id",
    "blocklists",
    "allowed_patterns",
    "max_payload_entropy",
    "drop_on_failure"
  ],
  "additionalProperties": false
}
```

## Considérations de Sécurité

La couche de filtrage à l'exécution agit comme un mécanisme supplémentaire de défense en profondeur. En s'exécutant avant la validation du schéma JSON de l'intention sémantique, elle réduit considérablement la surface d'attaque des injections avancées ciblant l'analyseur de validation lui-même ou intégrant des macros malveillantes dans des chaînes JSON par ailleurs conformes.

## Rétrocompatibilité

Cette RFC est entièrement rétrocompatible. Les systèmes mettant en œuvre le `v1.0.0-draft` de SDIC-1 peuvent déployer la couche de filtration à l'exécution comme un proxy en amont avant la couche de contrôle déterministe sans nécessiter de modifications de la sandbox de l'IA ou de l'Action Ledger.
