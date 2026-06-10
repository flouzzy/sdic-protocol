---
eip: 3
title: PRISM - Une architecture de routage sémantique inspirée du DNS pour les modèles de langage spécialisés
author: Mossaab Hassana Souaissa <mossaab.souaissa@gmail.com>
status: Draft
type: Standards Track
category: Core
created: 2026-06-10
---

[English](SDIC-RFC-0003.md) | [Français](SDIC-RFC-0003.fr.md)

## Résumé

Le déploiement industriel des grands modèles de langage (LLMs) révèle des limitations structurelles liées à leur architecture centralisée, notamment en ce qui concerne la légitimité des réponses, la responsabilité, le coût et la latence. Les systèmes actuels produisent une réponse même lorsqu'une requête dépasse leur véritable domaine d'expertise, faute de mécanisme de refus explicite ou de routage préalable.

Cet article propose un cadre architectural basé sur une infrastructure distribuée de modèles spécialisés, coordonnés par un mécanisme de routage sémantique inspiré du DNS. L'hypothèse centrale est que le routage de la connaissance doit précéder sa résolution.

Nous introduisons un protocole de cadrage contraignant, appelé clé-153, permettant la qualification, l'orientation et l'auto-évaluation des modèles, ainsi qu'un répertoire dynamique de compétences et une boucle de rétroaction empirique. Une machine à états formelle sépare strictement la décision de légitimité de l'exécution métier.

## Motivation

Les architectures LLM centralisées actuelles présentent plusieurs limitations structurelles : la production de réponses en dehors du véritable domaine de compétence, l'absence d'un mécanisme de refus natif, la difficulté à clarifier la légitimité d'une réponse, les coûts élevés associés à l'utilisation systématique de modèles généralistes et des niveaux de latence incompatibles avec certains cas d'utilisation industriels.

L'architecture proposée, PRISM (Protocol for Routed Intelligent Specialized Models), aborde ces limitations en routant une requête vers l'entité compétente avant toute résolution, séparant explicitement la décision de l'exécution. La connaissance doit être routée avant d'être résolue. Un système d'IA ne devrait pas tenter de répondre avant que les questions de légitimité, de portée et de niveau de compétence aient été explicitement abordées.

## L'analogie du centre de tri postal international

Imaginez un centre de tri postal international gérant des millions de colis chaque jour. Les colis arrivent avec des adresses, des tailles et des contenus variés. Si chaque travailleur essayait d'ouvrir chaque colis, de traiter son contenu et de le livrer lui-même, le système s'effondrerait dans le chaos, entraînant des erreurs de livraison et d'immenses retards. Au lieu de cela, le centre fonctionne avec des protocoles de routage stricts avant toute tentative de livraison.

Lorsqu'un colis entre dans l'installation, il passe d'abord par une station de numérisation préliminaire qui lit uniquement le code postal — elle n'ouvre pas le colis. Cette station consulte un répertoire dynamique de centres de livraison régionaux. Si le colis appartient à une région spécialisée, il est routé vers ce centre spécifique. Chaque centre a des règles strictes sur ce qu'il peut gérer (par exemple, un centre pour les objets fragiles refuse les machines lourdes). Ce n'est que lorsqu'il est confirmé qu'un colis se trouve au bon centre, le centre légitime, qu'il est ouvert et traité pour la livraison finale.

Dans notre architecture PRISM :

- **Les colis entrants** représentent les requêtes des utilisateurs, qui sont non structurées et couvrent une multitude de domaines.
- **La station de numérisation préliminaire** représente la console de routage centrale (VSLM). Elle analyse l'intention sémantique de la requête sans essayer d'y répondre, agissant comme un pré-résolveur.
- **Le répertoire dynamique de centres** correspond au 153-Registry, une base de données vivante référençant tous les modèles disponibles et leurs capacités explicites.
- **Les règles strictes des centres** incarnent le protocole de la clé-153, qui impose la qualification des modèles et exige un refus explicite si une requête sort de leur domaine.
- **Le traitement final pour la livraison** est la phase d'exécution métier, qui ne se produit qu'après que la décision de routage ait été formellement validée par la machine à états.

## Spécification

### 1. Console de routage centrale (VSLM)

La console centrale est implémentée comme un très petit modèle de langage (VSLM) contraint à n'effectuer que la classification des intentions. Elle agit comme un pré-résolveur, analogue à un serveur DNS, analysant l'intention de la requête, consultant le répertoire des compétences et prenant la décision de routage.

```text
+-----------+         +-------------+          +--------------+
| Utilisateur |       |    VSLM     |          | 153-Registry |
+-----+-----+         +------+------+          +-------+------+
      |                      |                         |
      | 1. Requête           |                         |
      +--------------------->|                         |
      |                      | 2. Analyser l'intention |
      |                      +------------------------>|
      |                      |                         | 3. Retourner l'ID du modèle
      |                      |<------------------------+
      |                      |                         |
      |                      | 4. Router la requête    |
      |                      +--------------------------------> [Modèle spécialisé]
```

### 2. Répertoire dynamique des compétences (153-Registry)

Le répertoire est une base de données vivante référençant tous les modèles disponibles. Chaque entrée est décrite par une carte de capacité. Aucun modèle n'est présumé être un expert par défaut.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RegistryEntry",
  "type": "object",
  "properties": {
    "model_id": {
      "type": "string",
      "description": "Identifiant unique pour le modèle spécialisé."
    },
    "domain": {
      "type": "string",
      "description": "Le domaine d'expertise principal."
    },
    "sub_domains": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Sous-domaines spécifiques gérés par le modèle."
    },
    "competence_score": {
      "type": "number",
      "minimum": 0,
      "maximum": 100,
      "description": "Score de compétence empirique."
    },
    "average_latency_ms": {
      "type": "number",
      "description": "Latence moyenne de réponse en millisecondes."
    },
    "refusal_rate": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Taux auquel le modèle refuse de manière appropriée les requêtes."
    },
    "status": {
      "type": "string",
      "enum": ["active", "inactive", "banned"],
      "description": "Statut actuel dans le répertoire."
    },
    "last_update": {
      "type": "string",
      "format": "date",
      "description": "Date de la dernière mise à jour des capacités."
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

### 3. La clé-153 : Qualification et exécution

La clé-153 est un protocole de cadrage contraignant imposant la qualification et l'exécution. La machine à états formelle sépare strictement la phase de décision de légitimité de la phase d'exécution métier.

```text
+-------------+          +-------------------+          +------------------+
|    VSLM     |          | Modèle spécialisé |          | Logique métier   |
+------+------+          +---------+---------+          +-------+----------+
       |                           |                            |
       | 1. Requête routée         |                            |
       +-------------------------->|                            |
       |                           | 2. Évaluer le domaine      |
       |                           |------------------+         |
       |                           |                  |         |
       |                           |<-----------------+         |
       |                           |                            |
       | 3a. REFUSED (Hors domaine)|                            |
       |<--------------------------+                            |
       |                           |                            |
       |                           | 3b. ACCEPTED (Dans domaine)|
       |                           +--------------------------->|
```

Toute réponse d'exécution DOIT adhérer au schéma suivant :

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Protocol153KeyResponse",
  "type": "object",
  "properties": {
    "status": {
      "type": "string",
      "enum": ["ACCEPTED", "REFUSED", "REJECTED"],
      "description": "Le statut de décision pour la requête."
    },
    "reason": {
      "type": "string",
      "maxLength": 100,
      "description": "Justification de l'acceptation ou du refus."
    },
    "response": {
      "type": "string",
      "description": "La réponse technique si acceptée, ou vide si refusée."
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

## Compatibilité descendante

Ce RFC étend le cadre SDIC-1 sans rompre les limites d'isolation déterministes existantes.
