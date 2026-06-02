---
rfc: 1
title: Standardisation du Registre d'Actions pour la Synchronisation Multi-Agents
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
created: 2026-06-02
---

[English](rfc-001-action-ledger-multi-agent-sync.md) | [Français](rfc-001-action-ledger-multi-agent-sync.fr.md)

## Résumé

Ce RFC propose une standardisation stricte de la structure de données du Registre d'Actions (Action Ledger) pour la synchronisation multi-agents au sein du protocole SDIC. La standardisation introduit une propagation déterministe de l'état via des intentions partagées en ajout seul, ainsi qu'une validation cryptographique des charges utiles canoniques. Cela garantit que les agents autonomes peuvent s'orchestrer de manière asynchrone sans muter directement l'état partagé, préservant ainsi l'intégrité déterministe du système hôte.

## Motivation

À mesure que les architectures multi-LLM évoluent, les agents doivent collaborer de manière fluide tout en respectant l'isolation cognitive imposée par SDIC-1. Les implémentations actuelles manquent d'un support standardisé permettant aux agents de diffuser et de valider des intentions en toute sécurité au sein d'un Consensus d'Essaim (Swarm Consensus). Les communications non structurées de type agent-à-agent présentent des risques de dérive d'état, de propagation d'injections de requêtes (prompt injection) et d'attaques par ré-étiquetage. Un schéma standardisé pour le Registre d'Actions garantit une coordination des agents immuable, déterministe et cryptographiquement vérifiée.

## Analogie de l'Orchestre Symphonique

Imaginez un grand orchestre symphonique interprétant une pièce complexe et non écrite. Chaque musicien est brillant et hautement créatif, capable de produire de belles mélodies. Cependant, ils ne peuvent pas simplement jouer quand ils le souhaitent, ni toucher directement l'instrument d'un autre musicien pour forcer la coordination. Au lieu de cela, ils regardent tous un chef d'orchestre central. Lorsqu'un musicien souhaite introduire une nouvelle harmonie, il écrit ses notes proposées sur un bout de papier et le remet au chef. Le chef examine le papier pour s'assurer que les notes sont lisibles et correspondent au rythme. Ce n'est que si les notes sont parfaites que le chef les copie sur la partition maîtresse, que chaque musicien lit ensuite pour ajuster son jeu.

Dans cette architecture, les brillants musiciens représentent les agents IA autonomes. Leur incapacité à toucher d'autres instruments représente l'isolation cognitive, empêchant toute mutation directe de l'état. Le bout de papier est l'intention sémantique, une proposition structurée d'action. Le chef d'orchestre agit comme la couche de contrôle déterministe et l'application hôte, vérifiant la proposition par rapport à un standard mathématique strict. Enfin, la partition maîtresse représente le Registre d'Actions, la source unique de vérité qui est mise à jour de manière déterministe et diffusée pour synchroniser l'ensemble de l'essaim.

## Spécification

Le registre de synchronisation multi-agents nécessite un Schéma JSON strict pour toutes les soumissions d'intentions. Ce schéma agit comme le `bout de papier` de l'analogie.

### Schéma d'Intention

Les agents doivent formater leurs propositions de synchronisation exactement selon le Schéma JSON suivant. Tous les objets doivent imposer `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentSyncIntent",
  "type": "object",
  "properties": {
    "agent_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$"
    },
    "correlation_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$"
    },
    "proposed_action": {
      "type": "string",
      "enum": ["read_state", "update_context", "delegate_task", "consensus_vote"]
    },
    "payload": {
      "type": "object",
      "properties": {
        "target_agent_id": { "type": "string" },
        "context_delta": { "type": "string" },
        "vote": { "type": "boolean" }
      },
      "additionalProperties": false
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "signature": {
      "type": "string"
    }
  },
  "required": ["agent_id", "correlation_id", "proposed_action", "payload", "timestamp", "signature"],
  "additionalProperties": false
}
```

### Signature Cryptographique et Canonisation

Pour garantir la non-répudiation et prévenir les attaques de ré-étiquetage, le champ `signature` doit être dérivé de la représentation canonique exacte de l'objet d'intention, en excluant strictement le champ `signature` lui-même.

1. **Canonisation :** L'objet JSON est dépouillé de la clé `signature`. La structure restante est sérialisée en utilisant un ordre des clés prévisible (par exemple, le tri lexicographique) et la suppression des espaces insignifiants (JSON Canonicalization Scheme - RFC 8785).
2. **Hachage :** Calculez le hachage SHA-256 de la chaîne UTF-8 canonisée.
3. **Signature :** L'agent signe le hachage en utilisant sa clé privée Ed25519 ou ECDSA.

### Diagramme de Séquence

```text
+----------+                     +-------------------+                  +---------------+
| Agent A  |                     | Filtre Validation |                  | Action Ledger |
+----------+                     +-------------------+                  +---------------+
     |                                     |                                    |
     | 1. Générer l'Intention Canonique    |                                    |
     |------------------------------------>|                                    |
     |                                     | 2. Vérifier la Conformité Schéma   |
     |                                     |--------------------------          |
     |                                     |                         |          |
     |                                     |<-------------------------          |
     |                                     |                                    |
     |                                     | 3. Validation Cryptographique      |
     |                                     |----------------------------        |
     |                                     |                           |        |
     |                                     |<---------------------------        |
     |                                     |                                    |
     |                                     | 4. Ajouter au Registre             |
     |                                     |----------------------------------->|
     |                                     |                                    |
     |                                     | 5. Diffuser la Mise à Jour d'État  |
     |<-------------------------------------------------------------------------|
     |                                     |                                    |
```

## Justification

Cette spécification limite strictement la communication inter-agents à un registre vérifié en ajout seul. L'application stricte du schéma (`"additionalProperties": false`) dissipe instantanément les intentions mal formées, annulant les tentatives d'injection de requêtes ciblant l'essaim. La vérification des signatures par rapport à une charge utile canonique garantit que les couches de routage intermédiaires ne peuvent pas manipuler les paramètres d'une action.

## Compatibilité Ascendante

Ce RFC étend le brouillon initial de SDIC-1 sans casser les implémentations existantes. Les configurations à un seul agent peuvent ignorer en toute sécurité `correlation_id` et les actions de coordination d'essaim, maintenant la compatibilité avec le concept de base du Registre d'Actions décrit dans la spécification racine.

## Considérations de Sécurité

Le principal vecteur d'attaque dans un Consensus d'Essaim est la pollution de contexte malveillante, où un agent compromis tente de propager des instructions hostiles à ses pairs. Ce risque est fortement atténué par :

1. Une validation de schéma stricte filtrant les données non structurées.
2. Une non-répudiation cryptographique liant chaque action à la clé publique d'un agent spécifique.
3. Des moteurs de transition d'état déterministes agissant comme les seules entités capables de modifier le contexte partagé.
