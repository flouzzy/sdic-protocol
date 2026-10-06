---
erc: 42
title: Schéma Standard pour l'Orchestration Asynchrone Multi-Agents (Consensus d'Essaim)
description: Définit la structure de données standard Intent_Proposal et le flux de consensus pour la synchronisation multi-agents.
author: Jules
status: Draft
type: Standards Track
created: 2026-10-15
license: CC0 1.0 Universal
---

## Résumé

Cette norme définit une architecture stricte et une structure de données pour la synchronisation des mutations d'état demandées par plusieurs agents autonomes dans le cadre de SDIC-1. Elle introduit le schéma `Intent_Proposal` et impose une application hôte centralisée et déterministe (le Chef d'Orchestre) pour valider, agréger et soumettre les propositions multi-agents à l'Action Ledger (Registre d'Actions) via le Consensus d'Essaim (Swarm Consensus).

## Motivation

Dans les architectures multi-agents, la communication directe inter-agents combinée à des résultats probabilistes introduit des risques graves de conditions de concurrence (race conditions), de dégradation intraçable de l'état et de vulnérabilités de routage. Un Essaim (Swarm) d'agents ne doit pas muter l'état partagé directement ni négocier les changements d'état de manière autonome. À la place, il doit exister un moyen mathématiquement solide et déterministe de parvenir à un consensus entre les agents et de persister ces décisions en toute sécurité dans l'Action Ledger.

### Analogie : La Session Parlementaire

Imaginez une session parlementaire où aucun représentant ne détient le pouvoir d'adopter une loi seul. Au lieu de cela, chaque représentant rédige indépendamment une proposition législative formelle et signée et la soumet directement à un président de l'assemblée strictement neutre. Le président ne débat pas ; il s'assure uniquement que chaque proposition correspond parfaitement au modèle constitutionnel. Ce n'est que lorsqu'un quorum de propositions valides et structurellement identiques est atteint que le président frappe le marteau, inscrivant de manière permanente la décision unifiée dans le registre légal officiel.

#### Correspondance Technique

- **Les Représentants :** Les Agents Autonomes opérant dans des bacs à sable (sandboxes) cognitifs isolés. Ils calculent et décident indépendamment.
- **La Proposition Législative Formelle :** Le `Intent_Proposal` (Proposition d'Intention). C'est une intention structurée représentant le vote individuel ou l'action demandée d'un agent.
- **Le Président de l'Assemblée Neutre :** Le Chef d'Orchestre (Couche de Contrôle Déterministe). Il applique des règles absolues et ne possède aucun raisonnement cognitif.
- **Le Modèle Constitutionnel :** Le schéma JSON strict et les mécanismes de vérification cryptographique. Si une proposition s'en écarte, elle est silencieusement rejetée.
- **Le Registre Légal Officiel :** L'Action Ledger. Il enregistre la transition d'état finale et déterministe uniquement après l'obtention du consensus.

## Spécification

Pour éviter les collisions asynchrones, les agents NE DOIVENT PAS exécuter de mutations directes sur les ressources partagées. Toutes les architectures multi-agents DOIVENT s'appuyer sur le mécanisme `Intent_Proposal`.

### Le Schéma Intent Proposal

Un `Intent_Proposal` représente la décision localisée d'un agent concernant une transition d'état partagée. Le schéma DOIT définir strictement toutes les propriétés requises et imposer `"additionalProperties": false` à tous les niveaux.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Intent_Proposal",
  "type": "object",
  "properties": {
    "proposal_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$",
      "description": "Identifiant unique de la session de consensus multi-agents."
    },
    "agent_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$",
      "description": "L'identifiant de l'agent proposant."
    },
    "proposed_state_mutation": {
      "type": "object",
      "properties": {
        "target_entity": { "type": "string" },
        "operation": { "type": "string", "enum": ["CREATE", "UPDATE", "DELETE", "MERGE"] },
        "payload": { "type": "string" }
      },
      "required": ["target_entity", "operation", "payload"],
      "additionalProperties": false
    },
    "confidence_score": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "cryptographic_signature": {
      "type": "string",
      "description": "La signature sur la représentation canonique de l'Intent_Proposal (à l'exclusion de ce champ de signature)."
    }
  },
  "required": [
    "proposal_id",
    "agent_id",
    "proposed_state_mutation",
    "confidence_score",
    "cryptographic_signature"
  ],
  "additionalProperties": false
}
```

### Le Flux de Travail du Consensus d'Essaim

Lorsque plusieurs agents participent à une session partagée, le Chef d'Orchestre (Couche de Contrôle Déterministe) facilite le consensus.

1. **Émission :** Chaque Agent $A_i$ génère un $Intent\_Proposal_i$ de manière isolée.
2. **Hachage Canonique et Signature :** Avant l'émission, l'agent sérialise le JSON (à l'exclusion de `cryptographic_signature`) en une chaîne canonique stricte, la hache et la signe avec sa Clé Privée.
3. **Vérification :** Le Chef d'Orchestre reçoit les propositions. Il valide d'abord le Schéma JSON (doit retourner exactement $1$ pour le succès). Ensuite, il vérifie la signature par rapport à la Clé Publique enregistrée de l'agent.
4. **Agrégation :** Le Chef d'Orchestre collecte les propositions vérifiées correspondant au même `proposal_id` sur une fenêtre temporelle définie.
5. **Évaluation du Consensus :** Si les conditions mathématiques du consensus (par exemple, $N > \text{Seuil}$ correspondant à la `proposed_state_mutation`) sont remplies, le Chef d'Orchestre génère un `Intent` déterministe final à soumettre à l'Action Ledger.
6. **Exécution :** La logique métier mute l'état.

#### Diagramme de Séquence UML

```text
+---------+         +---------+           +-------------------+            +---------------+
| Agent A |         | Agent B |           | Chef d'Orchestre  |            | Action Ledger |
+---------+         +---------+           +-------------------+            +---------------+
     |                   |                          |                              |
     | Intent_Proposal A |                          |                              |
     |--------------------------------------------->|                              |
     |                   |                          |                              |
     |                   | Intent_Proposal B        |                              |
     |                   |------------------------->|                              |
     |                   |                          |                              |
     |                   |                          | Valider Schéma JSON          |
     |                   |                          | Vérifier Signatures          |
     |                   |                          | Agréger & Consensus          |
     |                   |                          |----------------------------> |
     |                   |                          |                              |
     |                   |                          |                              | Commit Action
     |                   |                          |                              |
```

## Justification

Cette spécification garantit que la nature imprévisible de multiples agents IA interagissant est confinée mathématiquement. En transférant la responsabilité de la synchronisation de l'état des agents (probabiliste) au Chef d'Orchestre (déterministe), nous éliminons complètement les conditions de concurrence et la manipulation directe des données par des boucles agentiques non vérifiées. L'application de `"additionalProperties": false` supprime la surface d'attaque pour les injections de prompt tentant de faire passer des commandes non reconnues à travers la couche de consensus.

## Compatibilité Ascendante

Cet ERC est strictement une extension pour les capacités multi-agents et est entièrement rétrocompatible avec les implémentations mono-agent SDIC-1 v1.0.0-draft. Le nouveau schéma `Intent_Proposal` ne déprécie ni ne modifie la mécanique de base de l'Action Ledger ; il standardise simplement une couche de pré-calcul avant la soumission finale au registre.

## Droit d'auteur

Les droits d'auteur et les droits connexes sont renoncés via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
