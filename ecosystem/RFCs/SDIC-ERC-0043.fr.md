[Français](SDIC-ERC-0043.fr.md) | [English](SDIC-ERC-0043.md)

# SDIC-ERC-0043: Structure de Données du Registre d'Actions pour la Synchronisation Multi-Agents

- **Auteur:** Jules (AI Systems Engineer)
- **Statut:** Brouillon
- **Type:** Standards Track
- **Catégorie:** Architecture (Pilier IV)
- **Créé:** 2026-10-27

---

## Résumé

Cette norme définit un schéma strict et un protocole pour la structure de données du "Registre d'Actions" (Action Ledger) afin de faciliter l'orchestration et la synchronisation asynchrones multi-agents. En établissant un schéma JSON canonique et déterministe pour les intentions multi-agents et en tirant parti des signatures cryptographiques sur ces structures, cet ERC garantit que les agents peuvent coordonner les transitions d'état en toute sécurité sans accès direct les uns aux autres ou à l'état sous-jacent du système.

## 1. Motivation

Alors que les systèmes d'entreprise critiques adoptent de plus en plus des architectures multi-agents (Swarm), l'absence d'une structure de données standardisée pour la communication inter-agents et la synchronisation de l'état introduit des vulnérabilités importantes. Sans un registre strictement typé pour enregistrer les intentions des agents, le risque de dérive sémantique, de variables d'état hallucinées et d'exécution dictée par l'injection d'instructions augmente.

L'objectif de SDIC-ERC-0043 est de standardiser la charge utile du Registre d'Actions du Pilier IV spécifiquement pour le consensus de l'essaim (Swarm Consensus), en s'assurant que toute proposition de mutation d'état partagé doit passer par une couche de validation de schéma stricte et de confiance zéro avant d'être enregistrée et exécutée.

## 2. Analogie

Imaginez une salle de conseil d'administration d'entreprise de haute sécurité où plusieurs cadres (agents) doivent se mettre d'accord sur une décision financière majeure. Dans un système défectueux, ils pourraient simplement crier leurs idées à travers la pièce, ce qui entraînerait des malentendus, des manipulations ou des actions non autorisées par des participants malveillants.

Au lieu de cela, dans ce protocole strict, aucun cadre n'est autorisé à parler directement à un autre ou à la banque de l'entreprise. Lorsqu'un cadre souhaite proposer une action, il doit rédiger sa proposition sur un formulaire standardisé et inviolable. Ce formulaire nécessite des champs spécifiques (Action, Cible, Raison). Le cadre doit ensuite apposer physiquement sur le formulaire son sceau de cire unique et vérifiable (Signature Cryptographique). Ce formulaire est ensuite soumis à un greffier impartial et automatisé (Couche de Contrôle Déterministe). Le greffier vérifie uniquement si le formulaire est parfaitement rempli et si le sceau est valide. Si tel est le cas, la proposition est classée dans le registre public transparent (Registre d'Actions), où les autres cadres peuvent la lire et soumettre leurs propres formulaires en réponse.

### 2.1. Correspondance de l'Analogie

- **Les Cadres (Agents):** Les modèles d'IA probabilistes autonomes participant à l'essaim.
- **Crier à Travers la Pièce (Communication Directe):** Interactions directes non structurées et dangereuses entre LLM.
- **Le Formulaire Standardisé (Schéma du Registre d'Actions):** Le schéma JSON strict défini dans ce RFC, garantissant que toutes les propositions respectent un format structurel exact.
- **Le Sceau de Cire (Signature Cryptographique):** La preuve déterministe d'intention ($\sigma$), validant la forme canonique de la proposition.
- **Le Greffier Impartial (Couche de Contrôle Déterministe):** La logique de validation de l'application hôte, qui rejette les propositions non conformes ou mal signées.
- **Le Registre Public (Registre d'Actions):** L'historique temporel de l'état, en ajout uniquement, que les agents lisent pour synchroniser leur contexte.

## 3. Spécification

### 3.1. Schéma Standardisé du Registre d'Actions

Toutes les propositions de synchronisation multi-agents DOIVENT se conformer strictement au schéma JSON suivant. Afin d'empêcher les clés non définies, `"additionalProperties": false` DOIT être appliqué à tous les niveaux d'objets.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentSyncProposal",
  "type": "object",
  "properties": {
    "proposal_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique de la proposition."
    },
    "agent_id": {
      "type": "string",
      "description": "L'identifiant unique de l'agent qui propose."
    },
    "action_type": {
      "type": "string",
      "enum": ["propose_state_change", "endorse_proposal", "reject_proposal"],
      "description": "La nature de l'intention de l'agent."
    },
    "target_state_key": {
      "type": "string",
      "description": "La variable d'état ou la ressource spécifique ciblée."
    },
    "proposed_value": {
      "type": ["string", "number", "boolean", "null"],
      "description": "La nouvelle valeur souhaitée pour la clé d'état (le cas échéant)."
    },
    "justification": {
      "type": "string",
      "description": "Raisonnement sémantique derrière la proposition."
    },
    "reference_proposal_id": {
      "type": ["string", "null"],
      "format": "uuid",
      "description": "ID d'une proposition antérieure à laquelle cette action se rapporte (ex: pour les approbations)."
    },
    "signature": {
      "type": "string",
      "description": "Signature cryptographique sur la charge utile canonique."
    }
  },
  "required": [
    "proposal_id",
    "agent_id",
    "action_type",
    "target_state_key",
    "justification",
    "signature"
  ],
  "additionalProperties": false
}
```

### 3.2. Canonisation Cryptographique

Pour prévenir les attaques par rejeu et par ré-étiquetage au sein de l'essaim, la signature doit valider une représentation canonique de la proposition.

1. Créer un objet `CanonicalPayload` en supprimant le champ `signature` de la proposition.
2. Sérialiser `CanonicalPayload` de manière déterministe (par exemple, trier les clés par ordre lexicographique, supprimer les espaces).
3. Calculer la signature: $\sigma = Sign_{AgentPrivKey}(Hash(CanonicalPayload))$
4. Ajouter $\sigma$ à l'objet de proposition final en tant que champ `signature`.

### 3.3. Séquence d'Orchestration

La séquence d'opérations pour la synchronisation de l'état doit suivre strictement ce chemin déterministe.

```text
+----------+             +-------------------+             +-------------------+
| Agent A  |             | Couche de Contrôle|             | Registre d'Actions|
+----+-----+             +--------+----------+             +---------+---------+
     |                            |                                  |
     | 1. Générer le JSON         |                                  |
     | 2. Canoniser & Signer      |                                  |
     | 3. Soumettre la Charge     |                                  |
     +--------------------------->|                                  |
     |                            | 4. Valider le Schéma (Rejeter    |
     |                            |    si clés sup. existent)        |
     |                            | 5. Valider la Signature          |
     |                            | 6. Ajouter au Registre           |
     |                            +--------------------------------->|
     |                            |                                  |
     |                            |                                  |
+----+-----+                      |                                  |
| Agent B  |                      |                                  |
+----+-----+                      |                                  |
     | 7. Réhydrater le Contexte  |                                  |
     |<---------------------------+----------------------------------+
     | 8. Analyser la Proposition |                                  |
     | 9. Générer l'Approbation   |                                  |
     +--------------------------->|                                  |
     |                            | 10. Valider l'Approbation        |
     |                            | 11. Ajouter au Registre          |
     |                            +--------------------------------->|
```

## 4. Rétrocompatibilité

Cette norme est entièrement rétrocompatible avec les définitions fondamentales du Pilier IV décrites dans la v1.0.0-draft de la spécification SDIC-1. Elle étend le concept du Registre d'Actions en fournissant un schéma de charge utile spécialisé pour les environnements d'essaim sans modifier les exigences sous-jacentes du registre (telles que les identifiants de transaction ou les marqueurs temporels).

## 5. Licence

Ce document et son contenu sont publiés sous la licence CC0 1.0 Universel (Transfert dans le Domaine Public).
