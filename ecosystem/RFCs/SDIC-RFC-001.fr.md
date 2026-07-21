---
rfc: 1
title: Registre d'Actions Standardisé pour la Synchronisation Multi-Agents
author: Jules
status: Draft
type: Standards Track
created: 2026-07-21
---

[English](SDIC-RFC-001.md) | [Français](SDIC-RFC-001.fr.md)

## Résumé

Cette RFC propose une structure de données standardisée pour le "Registre d'Actions" (Action Ledger) au sein du framework SDIC-1, afin de faciliter la synchronisation multi-agents. Le Registre d'Actions fournit un enregistrement strict, en ajout seul (append-only), cryptographiquement vérifiable des intentions de l'IA. Cela garantit que dans un environnement multi-agents (Consensus d'Essaim), les intentions de divers agents autonomes peuvent être ordonnées, validées et synchronisées de manière déterministe sans mutation directe de l'état par un seul agent.

## Motivation

À mesure que les systèmes d'entreprise évoluent pour intégrer plusieurs agents autonomes, le besoin d'un mécanisme partagé et déterministement ordonné pour proposer des changements d'état devient primordial. Sans un Registre d'Actions standardisé, les agents pourraient proposer des actions conflictuelles, conduisant à des transitions d'état non déterministes et à des conditions de concurrence (race conditions). En standardisant strictement ce registre, nous permettons un Consensus d'Essaim robuste, garantissant que toutes les mutations d'état proposées sont enregistrées de manière immuable, validées structurellement et exécutées de manière déterministe.

## Spécification

### 1. Analogie style Feynman

Imaginez la cuisine animée d'un restaurant. Vous avez plusieurs cuisiniers, chacun travaillant sur des plats différents. Si chaque cuisinier jetait simplement ses ingrédients dans les marmites sur le feu quand bon lui semble, les repas seraient gâchés. Au lieu de cela, ils utilisent un rail de commandes.

Lorsqu'un cuisinier veut préparer quelque chose, il ne commence pas à cuisiner immédiatement. Il écrit exactement ce qu'il veut faire sur un bon de commande et le place sur le rail. Le chef de cuisine regarde les bons un par un. Le chef vérifie si le bon est rédigé dans le bon format et si les ingrédients sont disponibles. Si tout est correct, le chef donne l'ordre de préparer réellement le plat. Tous les cuisiniers peuvent voir le rail des bons, ils savent donc ce que tous les autres proposent et peuvent ajuster leurs propres plans en conséquence.

### 2. Correspondance Technique

Dans notre architecture technique, les composants de l'analogie correspondent directement à l'architecture SDIC-1 :

* **Les Cuisiniers (Agents) :** Ils représentent les agents IA autonomes indépendants opérant dans leurs environnements isolés (sandboxes). Ils calculent quelle action devrait se produire ensuite en fonction de leur contexte.
* **Les Marmites sur le Feu (État du Système) :** Il s'agit de la base de données réelle de l'application et de son état. Les agents n'ont aucun accès direct pour modifier cela.
* **Le Rail de Commandes (Le Registre d'Actions) :** C'est le registre standardisé, en ajout seul, où les agents soumettent leurs intentions. Il sert de source unique de vérité pour toutes les propositions de changements d'état.
* **Le Bon de Commande (Intention Sémantique) :** Il représente la charge utile JSON générée par l'agent, signée cryptographiquement et adhérant à un schéma strict.
* **Le Chef de Cuisine (Application Hôte / Validateur) :** Il s'agit de la couche de contrôle déterministe qui traite séquentiellement le registre, valide les schémas, vérifie les signatures cryptographiques et exécute les transitions d'état.

### 3. Définition du Schéma JSON

L'entrée du Registre d'Actions doit adhérer strictement au Schéma JSON suivant. Chaque intention d'agent doit être encapsulée dans une `intent_proposal`. Pour imposer une validation de schéma "Zero-Trust", `"additionalProperties": false` est strictement défini.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ActionLedgerEntry",
  "type": "object",
  "properties": {
    "transaction_id": {
      "type": "string",
      "format": "uuid"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "agent_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$"
    },
    "intent_proposal": {
      "type": "object",
      "properties": {
        "action_type": {
          "type": "string"
        },
        "payload": {
          "type": "object",
          "additionalProperties": false
        }
      },
      "required": ["action_type", "payload"],
      "additionalProperties": false
    },
    "cryptographic_signature": {
      "type": "string"
    }
  },
  "required": [
    "transaction_id",
    "timestamp",
    "agent_id",
    "intent_proposal",
    "cryptographic_signature"
  ],
  "additionalProperties": false
}
```

### 4. Diagramme de Séquence UML ASCII

Le diagramme de séquence suivant illustre la synchronisation du Consensus d'Essaim utilisant le Registre d'Actions.

```text
+---------+       +---------+       +---------------+       +---------------+
| Agent A |       | Agent B |       | Action Ledger |       | Host System   |
+---------+       +---------+       +---------------+       +---------------+
     |                 |                    |                       |
     |--- Intent 1 ------------------------>|                       |
     |                 |                    |                       |
     |                 |--- Intent 2 ------>|                       |
     |                 |                    |                       |
     |                 |                    |--- Read Ledger ------>|
     |                 |                    |                       |
     |                 |                    |<-- V(Intent 1) = 1 ---|
     |                 |                    |                       |
     |                 |                    |--- State Commit ----->|
     |                 |                    |                       |
     |<-- Context Sync (New State) ---------|                       |
     |                 |<-- Context Sync ---|                       |
```

### 5. Algorithme de Synchronisation Multi-Agents

La synchronisation suit un algorithme déterministe strict pour le consensus :

1. **Émission de l'Intention** : L'agent $A_i$ calcule une intention $I_i = C(X_i)$ en fonction de son contexte local $X_i$.
2. **Hachage Canonique** : L'agent crée une représentation JSON canonique de $I_i$, notée $I_{canon}$.
3. **Signature** : L'agent $A_i$ génère la signature $\sigma_i = Sign_{PrivKey_i}(Hash(I_{canon}))$.
4. **Soumission** : $A_i$ soumet $\{I_i, \sigma_i\}$ au Registre d'Actions.
5. **Validation Séquentielle** : Le Système Hôte lit les entrées en attente par ordre temporel. Pour chaque entrée $E$ :
    1. Vérifier $\sigma_i$ avec la Clé Publique de $A_i$.
    2. Si la signature est invalide, rejeter $E$.
    3. Vérifier le Schéma : $V(I_i)$. Si $V(I_i) = 0$, rejeter $E$.
    4. Évaluer la Logique Métier (Contraintes de consensus).
6. **Exécution de l'État** : Si l'étape 5 réussit, la transition d'état du système $S_{t+1} = E(S_t, I_i)$.
7. **Réhydratation** : Tous les agents récupèrent l'état mis à jour $S_{t+1}$ du registre pour rafraîchir leur contexte.

## Justification

Cette conception garantit que plusieurs agents ne peuvent pas créer de conditions de concurrence sur le système hôte déterministe. En forgeant tous les agents à écrire des propositions signées cryptographiquement dans un registre en ajout seul, le système hôte conserve l'autorité absolue sur la séquence et la validité des transitions d'état. Le schéma JSON strict garantit le déterminisme sémantique, rejetant toute charge utile hallucinatoire ou malformée avant qu'elle n'atteigne le moteur d'exécution.

## Rétrocompatibilité

Cette RFC introduit un nouveau standard de schéma pour les environnements multi-agents. Elle est entièrement rétrocompatible avec les systèmes mono-agent implémentant la version v1.0.0-draft, car un système mono-agent est fonctionnellement un système multi-agents avec un seul participant écrivant dans le registre.

## Considérations de Sécurité

Pour éviter les attaques de réétiquetage ou de rejeu (replay attacks), la signature cryptographique doit être calculée exclusivement sur la représentation canonique de l'entrée entière (à l'exclusion du champ de signature lui-même). L'inclusion de l'identifiant de transaction (`transaction_id` en UUIDv4) et de l'horodatage (`timestamp`) garantit la fraîcheur cryptographique de chaque soumission.

## Droit d'auteur

Droit d'auteur et droits connexes renoncés via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
