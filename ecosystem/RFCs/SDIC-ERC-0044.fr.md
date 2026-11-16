[Français](SDIC-ERC-0044.fr.md) | [English](SDIC-ERC-0044.md)

# SDIC-ERC-0044: Structure de données du registre d'actions pour la synchronisation multi-agents

- **Auteur:** Jules (Ingénieur Systèmes IA)
- **Statut:** Brouillon
- **Type:** Suivi des normes
- **Catégorie:** Architecture (Pilier IV)
- **Création:** 2026-11-01

---

## Résumé

Cette norme définit un schéma strict et un protocole pour la structure de données du "Registre d'Actions" (Action Ledger) afin de faciliter l'orchestration et la synchronisation asynchrones multi-agents. En établissant un schéma JSON canonique et déterministe pour les intentions multi-agents et en tirant parti des signatures cryptographiques sur ces structures, cet ERC garantit que les agents peuvent coordonner en toute sécurité les transitions d'état sans accès direct les uns aux autres ni à l'état du système sous-jacent.

## 1. Motivation

À mesure que les systèmes d'entreprise critiques adoptent de plus en plus des architectures multi-agents (Swarm), l'absence d'une structure de données standardisée pour la communication inter-agents et la synchronisation des états introduit des vulnérabilités importantes. Sans un registre strictement typé pour enregistrer les intentions des agents, le risque de dérive sémantique, de variables d'état hallucinées et d'exécution pilotée par l'injection de prompts augmente.

L'objectif du SDIC-ERC-0044 est de standardiser la charge utile du Registre d'Actions du Pilier IV spécifiquement pour le consensus Swarm, garantissant que toute proposition de mutation d'état partagé doit passer par une couche stricte de validation de schéma Zero-Trust avant d'être enregistrée et traitée.

## 2. Analogie

Imaginez une salle de conseil d'entreprise de haute sécurité où plusieurs cadres (agents) doivent s'entendre sur une décision financière majeure. Dans un système défaillant, ils pourraient simplement crier leurs idées à travers la pièce, entraînant des malentendus, des manipulations ou des actions non autorisées de la part de participants malveillants.

Au lieu de cela, dans ce protocole strict, aucun cadre n'est autorisé à parler directement à un autre ou à la banque de l'entreprise. Lorsqu'un cadre souhaite proposer une action, il doit rédiger sa proposition sur un formulaire papier carbone standardisé et infalsifiable. Ce formulaire nécessite des champs spécifiques (Action, Cible, Raison). Le cadre doit ensuite apposer physiquement sur le formulaire son sceau de cire unique et vérifiable (Signature Cryptographique). Ce formulaire est ensuite soumis à un greffier automatisé et impartial (Couche de Contrôle Déterministe). Le greffier vérifie uniquement si le formulaire est parfaitement rempli et si le sceau est valide. Si c'est le cas, la proposition est déposée dans le registre public transparent (Registre d'Actions), où les autres cadres peuvent la lire et soumettre leurs propres formulaires en réponse.

### 2.1. Cartographie de l'analogie

- **Les Cadres (Agents) :** Les modèles d'IA probabilistes autonomes participant à l'essaim.
- **Crier à travers la pièce (Communication directe) :** Interactions LLM-à-LLM non structurées et dangereuses.
- **Le formulaire papier carbone standardisé (Schéma du Registre d'Actions) :** Le schéma JSON strict défini dans ce RFC, garantissant que toutes les propositions adhèrent à un format structurel exact.
- **Le sceau de cire (Signature cryptographique) :** La preuve d'intention déterministe ($\sigma$), validant la forme canonique de la proposition.
- **Le greffier impartial (Couche de contrôle déterministe) :** La logique de validation de l'application hôte, qui rejette les propositions non conformes ou mal signées.
- **Le registre public (Registre d'actions) :** L'historique des états temporel, en ajout seul, que les agents lisent pour synchroniser leur contexte.

## 3. Spécification

### 3.1. Schéma standardisé du Registre d'Actions

Toutes les propositions de synchronisation multi-agents DOIVENT se conformer strictement au schéma JSON suivant. Pour empêcher les clés non définies, `"additionalProperties": false` DOIT être appliqué à tous les niveaux d'objet.

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
      "description": "L'identifiant unique de l'agent proposant."
    },
    "action_type": {
      "type": "string",
      "enum": ["propose_state_change", "endorse_proposal", "reject_proposal"],
      "description": "La nature de l'intention de l'agent."
    },
    "target_state_key": {
      "type": "string",
      "description": "La variable d'état ou ressource spécifique ciblée."
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
      "description": "ID d'une proposition antérieure à laquelle cette action se rapporte (par ex. pour les approbations)."
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

### 3.2. Canonicalisation cryptographique

Pour prévenir les attaques par rejeu et par réétiquetage au sein de l'essaim, la signature doit valider une représentation canonique de la proposition.

1. Créer un objet `CanonicalPayload` en retirant le champ `signature` de la proposition.
2. Sérialiser `CanonicalPayload` de manière déterministe (ex. trier les clés lexicographiquement, supprimer les espaces blancs).
3. Calculer la signature : $\sigma = Sign_{AgentPrivKey}(Hash(CanonicalPayload))$
4. Ajouter $\sigma$ à l'objet proposition final en tant que champ `signature`.

### 3.3. Séquence d'orchestration

La séquence d'opérations pour la synchronisation d'état doit suivre strictement ce chemin déterministe.

```text
+----------+             +-----------------+             +---------------+
| Agent A  |             | Couche de C.    |             | Registre Act. |
+----+-----+             +--------+--------+             +-------+-------+
     |                            |                              |
     | 1. Générer JSON Propos.    |                              |
     | 2. Canonicaliser & Signer  |                              |
     | 3. Soumettre Charge        |                              |
     +--------------------------->|                              |
     |                            | 4. Valider Schéma (Rejeter   |
     |                            |    si clés extra existent)   |
     |                            | 5. Valider Signature         |
     |                            | 6. Ajouter au Registre       |
     |                            +----------------------------->|
     |                            |                              |
     |                            |                              |
+----+-----+                      |                              |
| Agent B  |                      |                              |
+----+-----+                      |                              |
     | 7. Réhydrater Contexte     |                              |
     |<---------------------------+------------------------------+
     | 8. Analyser Nv. Propos.    |                              |
     | 9. Générer Approbation     |                              |
     +--------------------------->|                              |
     |                            | 10. Valider Approbation      |
     |                            | 11. Ajouter au Registre      |
     |                            +----------------------------->|
```

## 4. Compatibilité descendante

Cette norme est entièrement rétrocompatible avec les définitions fondamentales du Pilier IV décrites dans la version v1.0.0-draft de la spécification SDIC-1. Elle étend le concept du registre d'actions en fournissant un schéma de charge utile spécialisé pour les environnements Swarm sans modifier les exigences sous-jacentes du registre (telles que les identifiants de transaction ou les marqueurs temporels).

## 5. Licence

Ce document et son contenu sont publiés sous la licence CC0 1.0 Universelle (Transfert dans le Domaine Public).
