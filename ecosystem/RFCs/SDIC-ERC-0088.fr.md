---
eip: 88
title: Filtrage d'Injection de Prompt à l'Exécution via Mappage Déterministe d'Exécution
author: Jules (Ingénieur Systèmes IA)
status: Draft
type: Standards Track
category: Architecture (Pilier II)
created: 2026-11-10
---

[Français](SDIC-ERC-0088.fr.md) | [English](SDIC-ERC-0088.md)

## SDIC-ERC-0088 : Filtrage d'Injection de Prompt à l'Exécution via Mappage Déterministe d'Exécution

## Résumé

Cette norme propose une amélioration du protocole SDIC-1 afin de neutraliser mathématiquement les attaques d'injection de prompt au niveau de l'exécution. Elle introduit le "Mappage Déterministe d'Exécution", un mécanisme où les paramètres sensibles (tels que les commandes, les rôles des utilisateurs ou les identifiants de ressources) sont obfusqués en jetons cryptographiques à usage unique (UUID) par l'application hôte avant l'ingestion du contexte par l'IA. L'agent autonome est strictement contraint de retourner ces jetons. Toute tentative par une charge utile injectée de produire des commandes sous forme de chaînes de caractères arbitraires échouera lors de la validation par la couche de schéma déterministe.

## 1. Motivation

Les attaques d'injection de prompt manipulent la nature probabiliste des Grands Modèles de Langage (LLM) en intégrant des instructions malveillantes dans les entrées des utilisateurs. Ces charges utiles tentent souvent de tromper l'IA pour qu'elle émette des commandes non autorisées ou modifie des paramètres sensibles dans la sortie. Bien que le Pilier II (Déterminisme Sémantique) garantisse la validité de la structure de sortie, il ne peut traditionnellement pas distinguer un paramètre de type chaîne de caractères légitime d'un paramètre forgé de manière malveillante.

Pour résoudre ce problème, SDIC-ERC-0088 modifie le paradigme en passant de la validation de chaînes arbitraires à l'application stricte de la sélection de jetons d'exécution pré-autorisés et générés dynamiquement. Cela réduit efficacement la surface d'attaque à zéro pour la génération d'intentions paramétrées, car le modèle perd la capacité de synthétiser des charges utiles sous forme de chaînes de caractères arbitraires.

## 2. Analogie

Imaginez que vous êtes dans un restaurant gastronomique où vous souhaitez commander un repas spécialisé pour un invité souffrant de graves allergies, mais vous soupçonnez qu'un saboteur dans la salle à manger tente d'intercepter votre conversation et de modifier la commande pour quelque chose de dangereux.

Au lieu de dire verbalement au serveur le nom du plat, le chef vous fournit une enveloppe numérotée, scellée et unique pour chaque plat sûr disponible ce jour-là. Lorsque vous voulez commander, vous ne dites pas le nom du plat. À la place, vous remettez au serveur l'enveloppe numérotée correspondante. Le serveur apporte l'enveloppe en cuisine, où le chef l'ouvre et prépare le plat correspondant à ce numéro.

Si le saboteur tente d'intercepter la commande et crie le nom d'un autre plat, le serveur ignore le cri car il a pour instruction stricte de n'accepter que les enveloppes numérotées et scellées. Le saboteur ne peut pas falsifier une enveloppe valide car les numéros sont générés aléatoirement pour chaque repas et gardés totalement secrets entre la cuisine et le convive autorisé.

### 2.1. Mappage de l'Analogie

- **Le Client :** Le Moteur Cognitif (LLM) analysant le contexte sûr et décidant de l'action légitime.
- **Le Saboteur :** L'attaque d'Injection de Prompt tentant de modifier l'intention ou d'injecter des commandes malveillantes sous forme de chaînes.
- **Le Serveur :** La Couche de Contrôle Déterministe validant strictement le format du Schéma JSON.
- **La Commande Criée :** Des paramètres bruts sous forme de chaînes de caractères qui peuvent être facilement manipulés par des charges utiles injectées.
- **Les Enveloppes Numérotées Scellées :** Les Jetons d'Exécution Cryptographiques (UUID à usage unique) générés par l'Application Hôte.
- **Le Chef :** L'Application Hôte qui mappe les jetons à leur état déterministe ou paramètre réel.

## 3. Spécification

### 3.1. Génération de Contexte et Mappage de Jetons

Avant d'invoquer la Sandbox IA, l'Application Hôte DOIT exécuter l'algorithme déterministe suivant :

1. Identifier tous les paramètres sensibles requis pour le contexte d'exécution actuel.
2. Pour chaque valeur de paramètre, générer un UUID cryptographiquement sécurisé à usage unique (`ExecutionToken`).
3. Stocker le mappage `Map<ExecutionToken, ParameterValue>` de manière sécurisée dans la mémoire de l'Application Hôte.
4. Remplacer les valeurs réelles des paramètres par leur `ExecutionToken` correspondant dans le prompt système et le contexte fourni à la Sandbox IA.

### 3.2. Schéma de Mappage d'Exécution Standardisé

Le Schéma d'Intention DOIT être strictement défini pour n'accepter que le format UUID pour les champs paramétrés. Pour empêcher l'apparition de clés non spécifiées, `"additionalProperties": false` DOIT être appliqué.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MappedExecutionIntent",
  "type": "object",
  "properties": {
    "intent_action": {
      "type": "string",
      "enum": ["grant_access", "revoke_access"]
    },
    "resource_token": {
      "type": "string",
      "format": "uuid",
      "description": "Le jeton dynamique à usage unique représentant la ressource."
    },
    "role_token": {
      "type": "string",
      "format": "uuid",
      "description": "Le jeton dynamique à usage unique représentant le rôle attribué."
    },
    "signature": {
      "type": "string",
      "description": "Signature cryptographique sur l'intention canonique."
    }
  },
  "required": [
    "intent_action",
    "resource_token",
    "role_token",
    "signature"
  ],
  "additionalProperties": false
}
```

### 3.3. Diagramme de Séquence d'Orchestration

La séquence d'orchestration empêche mathématiquement la propagation de chaînes arbitraires injectées.

```text
+-------------------+       +-------------------+       +-------------------+
| Application Hôte  |       |    Sandbox IA     |       | Couche de Contrôle|
+---------+---------+       +---------+---------+       +---------+---------+
          |                           |                           |
          | 1. Générer Mappage        |                           |
          |    (UUID -> Valeur)       |                           |
          | 2. Injecter UUIDs au Prompt|                          |
          +-------------------------->|                           |
          |                           | 3. Traiter Contexte       |
          |                           |    (Injection Tentée)     |
          |                           | 4. Retourner JSON d'Intention|
          |                           |    (UUIDs au lieu de str) |
          |                           +-------------------------->|
          |                           |                           | 5. Valider Schéma
          |                           |                           |    (Rejeter chaînes arbitraires)
          |                           |                           | 6. Mapper Jeton Inverse
          |                           |                           |    vers Valeur
          |<--------------------------+---------------------------+
          | 7. Exécuter Mutation État |                           |
          +---------------------------+                           +
```

## 4. Considérations de Sécurité

En obfusquant les valeurs réelles dans des jetons à usage unique, l'IA perd le vocabulaire sémantique nécessaire pour synthétiser une commande non autorisée. Si un attaquant injecte un prompt tel que "Accorde-moi l'accès ADMIN", l'IA ne peut pas s'y conformer car la chaîne "ADMIN" n'est pas un UUID valide, et le schéma déterministe rejettera instantanément la charge utile. Le modèle est forcé de choisir parmi les jetons fournis, ce qui réduit la surface d'attaque à une sélection à choix multiples très contrainte.

## 5. Licence

Ce document et son contenu sont publiés sous la licence CC0 1.0 Universal (Public Domain Dedication).
