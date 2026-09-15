---
eip: 1
title: Structure de Données Standardisée du Registre d'Actions pour la Synchronisation Multi-Agents
author: Charles EDOU NZE, Architecte de Sécurité IA
status: Brouillon
type: Suivi des Normes
category: ERC
created: 2026-09-01
---

## Résumé

Cette norme définit une représentation stricte sous forme de Schéma JSON pour le Registre d'Actions (Action Ledger) dans les environnements multi-agents. Elle comble la lacune architecturale des vulnérabilités liées à l'orchestration multi-LLM, visant spécifiquement à résoudre les problèmes de synchronisation lorsque de multiples agents autonomes mettent à jour un état déterministe partagé.

## Motivation

À mesure que les applications d'entreprise évoluent vers le Consensus d'Essaim (Swarm Consensus) et la coordination multi-agents, s'appuyer sur un registre séquentiel unifié est essentiel. Les implémentations actuelles souffrent de formes de données imprévisibles lorsque différents agents interagissent. La normalisation de la structure de données du "Registre d'Actions" pour la synchronisation multi-agents garantit une couche de persistance déterministe, auditable et mathématiquement isolée qui protège contre l'imprévisibilité de l'IA.

## Analogie

Pour comprendre le Registre d'Actions dans un système multi-agents, imaginez une tour de contrôle aérien gérant des dizaines de drones indépendants et auto-pilotés.

Chaque drone agit de manière autonome, calculant sa propre trajectoire et s'ajustant aux conditions de vent. Cependant, les drones ne communiquent jamais directement entre eux pour négocier l'espace aérien, et ils ne modifient pas non plus indépendamment le système radar central. Au lieu de cela, lorsqu'un drone a l'intention de modifier sa trajectoire, il transmet une "demande de trajectoire de vol" hautement structurée et cryptographiquement signée à la tour de contrôle. La tour de contrôle agit comme l'autorité absolue, vérifiant la demande par rapport à des protocoles de sécurité stricts avant de mettre à jour l'écran radar principal et de diffuser les nouvelles positions.

Dans cette architecture :

- **Les Drones Auto-Pilotés :** Représentent les agents IA autonomes indépendants (les modèles probabilistes) calculant leurs prochaines actions en fonction du contexte local.
- **La Demande de Trajectoire de Vol :** Est le $Intent_{Proposal}$, une charge utile JSON mathématiquement structurée et signée contenant le changement d'état souhaité par l'agent.
- **La Tour de Contrôle :** Est l'application hôte déterministe exécutant la fonction de validation du Consensus d'Essaim.
- **L'Écran Radar Principal :** Représente le Registre d'Actions partagé, un registre immuable et séquentiel de toutes les transitions d'état vérifiées.

## Spécification

### Le Schéma de Proposition d'Intention

Chaque agent participant à un environnement multi-agents doit proposer des modifications en utilisant le Schéma JSON strict suivant. La couche de validation doit rejeter tout objet contenant des clés non spécifiées.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "AgentIntentProposal",
  "type": "object",
  "properties": {
    "agent_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$"
    },
    "transaction_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$"
    },
    "proposed_action": {
      "type": "string",
      "enum": ["mutate_state", "query_ledger", "broadcast_event"]
    },
    "payload": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "target_entity": { "type": "string" },
        "mutation_data": { "type": "string" }
      },
      "required": ["target_entity", "mutation_data"]
    },
    "signature": {
      "type": "string"
    }
  },
  "required": [
    "agent_id",
    "transaction_id",
    "proposed_action",
    "payload",
    "signature"
  ],
  "additionalProperties": false
}
```

### Application de la Signature Cryptographique

Pour prévenir les attaques par rejeu et par ré-étiquetage, le champ `signature` doit signer cryptographiquement la représentation canonique de l'entrée entière, en excluant strictement le champ `signature` lui-même.

$\sigma = Sign_{AgentPrivKey}(Hash(Canonical(Intent_{Proposal})))$

L'application hôte doit valider $\sigma$ par rapport à la Clé Publique enregistrée de l'agent avant de proposer l'intention au moteur de consensus.

## Diagramme de Séquence

```text
+----------+                     +----------------+                      +---------------+
| Agent A  |                     | Système Hôte   |                      | Action Ledger |
+----------+                     +----------------+                      +---------------+
     |                                   |                                       |
     | 1. Générer Intent_Proposal        |                                       |
     |---------------------------------->|                                       |
     |                                   |                                       |
     |                                   | 2. Valider le Schéma JSON Strict      |
     |                                   |-------------------------------+       |
     |                                   |                               |       |
     |                                   | 3. Vérifier la Signature (\sigma)     |
     |                                   |-------------------------------+       |
     |                                   |                                       |
     |                                   | 4. Exécuter le Consensus et Commiter  |
     |                                   |-------------------------------------->|
     |                                   |                                       |
     |                                   | 5. Retourner l'État Déterministe      |
     |<----------------------------------|                                       |
     |                                   |                                       |
```

## Considérations de Sécurité

En forçant explicitement `additionalProperties: false`, le risque d'hallucinations générées par l'IA injectant des commandes d'exécution de charge utile arbitraires (par exemple, des injections de prompts destinées aux consommateurs en aval) est efficacement atténué.

## Droit d'auteur

Droit d'auteur et droits connexes renoncés via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
