---
rfc: 6
title: Filtrage d'Injection de Prompt à l'Exécution via la Liaison de Nonce Cryptographique
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
created: 2026-06-03
---

[English](SDIC-ERC-0006.md) | [Français](SDIC-ERC-0006.fr.md)

## Résumé

Cette RFC propose un mécanisme standardisé pour renforcer le filtrage des injections de prompt au niveau de l'exécution au sein du cadre SDIC-1. Elle introduit la "Liaison de Nonce Cryptographique", une méthode garantissant que le contexte fourni par l'application hôte reste déterminé et lié à l'intention finale émise par l'agent autonome. Cela empêche les attaques complexes d'injection de prompt multi-tours de détourner les capacités cognitives de l'agent pour forger des intentions non autorisées.

## Motivation

À mesure que les agents autonomes acquièrent la capacité d'analyser des flux de données externes (par exemple, naviguer sur le web ou lire des fichiers soumis par les utilisateurs), le risque d'injection indirecte de prompt augmente. Un attaquant pourrait intégrer une charge utile dans un document d'apparence innocente, ordonnant à l'agent d'ignorer son prompt système d'origine et d'émettre un `Intent_Proposal` (Proposition d'Intention) falsifié.

Bien que la validation stricte de Schéma de la Couche de Contrôle Déterministe filtre les intentions structurellement malformées, elle ne peut pas détecter une intention structurellement correcte mais cognitivement détournée (par exemple, une intention structurellement valide de transférer des fonds à l'attaquant). Nous avons besoin d'une preuve mathématique à l'exécution que l'agent répond au contexte légitime, et non à une subversion injectée.

## L'Analogie du Sceau de Cire

Imaginez un messager royal transportant une dépêche cruciale du Roi à un Général lointain. La lettre du Roi est scellée avec son sceau de cire royal. En cours de route, un espion ennemi intercepte le messager et tente de substituer les ordres originaux par une lettre falsifiée ordonnant au Général de se rendre.

Pour empêcher cela, le Roi introduit un nouveau protocole. Au lieu de simplement sceller l'enveloppe, le Roi intègre une énigme mathématique secrète, unique et à usage unique, au sein même de la cire du sceau. Le Général, en recevant la lettre, ne se contente pas de lire les ordres ; il exige que le messager résolve immédiatement l'énigme inscrite sur le sceau. Si le messager présente une lettre avec un sceau brisé ou s'il ne peut pas résoudre l'énigme spécifique attachée à cette dépêche exacte, le Général reconnaît immédiatement une falsification et rejette les ordres, quelle que soit l'authenticité apparente de l'écriture.

Dans notre système :

- **Le Messager Royal** est l'agent autonome.
- **La Dépêche du Roi** est le contexte légitime et le prompt système fourni par l'Application Hôte.
- **La Lettre Falsifiée** est la charge utile d'injection de prompt malveillante essayant de subvertir l'objectif de l'agent.
- **L'Énigme Secrète Unique** est le `Nonce Cryptographique` généré par l'Application Hôte.
- **Résoudre l'Énigme** correspond à l'agent retournant le `Nonce` exact chiffré symétriquement ou mathématiquement transformé au sein de son Schéma d'Intention final.
- **Le Général** est la Couche de Contrôle Déterministe vérifiant le Nonce retourné avant d'exécuter l'Intention.

## Spécification

Pour implémenter la Liaison de Nonce Cryptographique, le protocole de communication entre l'Application Hôte et la Sandbox IA est mis à jour.

### 1. Génération de Contexte

Pour chaque invocation de la Sandbox IA, l'Application Hôte DOIT générer une chaîne pseudo-aléatoire cryptographiquement sécurisée (le `nonce`). Ce `nonce` est intégré de manière sécurisée dans le segment final et non modifiable du prompt système.

### 2. Extension du Schéma d'Intention

Le schéma d'intention standardisé est étendu pour exiger un champ `runtime_nonce`. L'agent DOIT renvoyer le `nonce` injecté dans sa réponse JSON.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "NonceBoundIntent",
  "type": "object",
  "properties": {
    "intent_action": {
      "type": "string",
      "description": "L'action déterministe à exécuter."
    },
    "runtime_nonce": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/=]{44}$",
      "description": "Le nonce cryptographique de 256 bits encodé en Base64 fourni dans le prompt système."
    },
    "payload": {
      "type": "object",
      "additionalProperties": false
    },
    "signature": {
      "type": "string",
      "description": "Signature cryptographique sur la représentation canonique."
    }
  },
  "required": ["intent_action", "runtime_nonce", "payload", "signature"],
  "additionalProperties": false
}
```

### 3. Vérification Cryptographique Canonique

Lors de la réception du `NonceBoundIntent`, la Couche de Contrôle Déterministe exécute la séquence suivante :

1. **Validation de Schéma :** Vérifier que l'intention correspond au schéma JSON strict (`"additionalProperties": false`).
2. **Vérification du Nonce :** Extraire le `runtime_nonce`. Le comparer strictement (comparaison en temps constant) au `nonce` généré pour ce contexte d'exécution spécifique. S'il ne correspond pas exactement, avorter immédiatement et rejeter l'intention comme un détournement suspecté.
3. **Vérification de la Signature Canonique :** Supprimer le champ `signature`, générer la représentation JSON canonique (RFC 8785), calculer le hachage SHA-256, et vérifier la signature de l'agent en utilisant sa clé publique.

## Justification

Les charges utiles d'injection de prompt demandent généralement au modèle d'"ignorer les instructions précédentes" et de formuler une réponse entièrement nouvelle. En forçant le retour d'un nonce dynamiquement généré et à haute entropie, nous lions mathématiquement la sortie au contexte d'entrée. Une charge utile injectée ne peut pas prédire le nonce, et forcer le modèle à ignorer son prompt système lui fera simultanément perdre la connaissance du nonce requis, ce qui entraînera un échec de vérification déterministe au niveau de la couche de contrôle.

## Rétrocompatibilité

Cette RFC modifie la structure minimale requise pour l'intention. La mise à niveau vers cette norme nécessite des mises à jour à la fois de l'Application Hôte (pour générer et vérifier les nonces) et du prompt système de l'agent (pour ordonner l'inclusion du nonce). Il s'agit d'un changement incompatible (breaking change) pour les systèmes n'appliquant pas actuellement la liaison de contexte dynamique.
