---
eip: 12
title: Standardisation de la structure de données du Registre d'Actions pour la synchronisation multi-agents
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
category: Core
created: 2026-09-22
---

[English](SDIC-ERC-0012.md) | [Français](SDIC-ERC-0012.fr.md)

## Résumé

Cette norme définit un protocole déterministe strict et une structure de données pour la synchronisation des systèmes multi-agents au sein du framework SDIC-1. Elle standardise la structure de données du Registre d'Actions (Action Ledger) pour s'assurer que les agents autonomes peuvent proposer des changements d'état en toute sécurité, en s'appuyant sur des signatures cryptographiques et une validation stricte du schéma JSON sans aucune propriété non spécifiée autorisée.

## Motivation

À mesure que les systèmes d'IA d'entreprise évoluent pour intégrer plusieurs agents autonomes, le risque de dégradation de l'état se multiplie. Les agents opérant de manière probabiliste pourraient tenter de muter l'état partagé simultanément ou d'injecter des champs non suivis. Pour prévenir ces vulnérabilités, la synchronisation multi-agents ne doit pas reposer sur l'autorégulation des agents. Elle doit plutôt s'appuyer sur un Registre d'Actions déterministe et mathématiquement vérifiable qui oblige les agents à soumettre des propositions standardisées et signées cryptographiquement.

## L'Analogie de l'Orchestre Symphonique

Imaginez un grand orchestre symphonique interprétant un morceau complexe. Chaque musicien brillant représente un agent autonome. Si chaque musicien joue simplement quand il le souhaite, le résultat est une cacophonie. De plus, aucun musicien n'est autorisé à tendre la main et à jouer de l'instrument d'un autre musicien.

Pour créer de la musique, chaque musicien doit écrire ses notes proposées sur un bout de papier standardisé et le remettre à un chef d'orchestre central. Le chef d'orchestre agit comme un applicateur absolu des règles, vérifiant que le papier est parfaitement formaté, écrit lisiblement et signé par le musicien qui l'a soumis. Si un papier contient des gribouillis supplémentaires non demandés, le chef d'orchestre le jette immédiatement. S'il est parfait, le chef d'orchestre le transcrit sur la partition maîtresse. L'ensemble de l'orchestre lit ensuite cette partition maîtresse pour ajuster ses prochaines notes.

Dans l'architecture SDIC-1 :

- **Les Musiciens :** Les agents IA autonomes opérant de manière probabiliste.
- **La Restriction d'Instrument :** L'Isolation Cognitive (aucune mutation directe de l'état ni d'accès réseau).
- **Le Bout de Papier :** L'intention sémantique, formatée comme un objet JSON strictement typé.
- **La Vérification du Chef d'Orchestre :** La Couche de Contrôle Déterministe appliquant une validation stricte du schéma et une vérification de la signature cryptographique.
- **Le Papier Jeté :** Le rejet des injections de prompt ou des hallucinations en appliquant `"additionalProperties": false`.
- **La Partition Maîtresse :** Le Registre d'Actions, agissant comme l'état de synchronisation déterministe et en ajout seul (append-only).

## Spécification

Le registre de synchronisation multi-agents nécessite un Schéma JSON strict pour toutes les soumissions d'intentions. Ce schéma agit comme le bout de papier standardisé.

### Schéma d'Intention

Les agents DOIVENT formater leurs propositions de synchronisation exactement selon le Schéma JSON suivant. Tous les objets DOIVENT appliquer `"additionalProperties": false` pour neutraliser instantanément les charges utiles d'injection de prompt tentant d'ajouter des données non validées.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentSyncIntent",
  "type": "object",
  "properties": {
    "agent_id": {
      "type": "string",
      "format": "uuid"
    },
    "correlation_id": {
      "type": "string",
      "format": "uuid"
    },
    "proposed_action": {
      "type": "string",
      "enum": ["read_state", "update_context", "delegate_task", "consensus_vote"]
    },
    "payload": {
      "type": "object",
      "properties": {
        "target_agent_id": { "type": "string", "format": "uuid" },
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

### Identité Cryptographique & Preuve d'Intention

Pour garantir la non-répudiation et prévenir les attaques par ré-étiquetage, le champ `signature` DOIT être dérivé de la représentation canonique exacte de l'objet d'intention, en excluant strictement le champ `signature` lui-même.

1. **Canonisation :** L'objet JSON est dépouillé de la clé `signature`. La structure restante est sérialisée en utilisant un ordre de clés prévisible (tri lexicographique) et la suppression des espaces non significatifs (Schéma de Canonisation JSON).
2. **Hachage :** Calculer le hachage SHA-256 de la chaîne UTF-8 canonisée.
3. **Signature :** L'agent signe le hachage en utilisant sa clé privée désignée.
4. **Vérification :** La Couche de Contrôle Déterministe vérifie la signature par rapport à la clé publique connue de l'agent.

### Diagramme de Séquence

L'interaction entre un agent et le Registre d'Actions déterministe est strictement asynchrone et validée par l'application hôte.

```text
+----------+                     +-----------------------+                  +---------------+
| Agent A  |                     | Contrôle Déterministe |                  | Action Ledger |
+----------+                     +-----------------------+                  +---------------+
     |                                     |                                    |
     | 1. Générer Intention Canonique      |                                    |
     |------------------------------------>|                                    |
     |                                     | 2. Vérifier Conformité Schéma      |
     |                                     | (Rejeter si invalide)              |
     |                                     |                                    |
     |                                     | 3. Validation Cryptographique      |
     |                                     | (Rejeter si signature invalide)    |
     |                                     |                                    |
     |                                     | 4. Ajouter au Registre             |
     |                                     |----------------------------------->|
     |                                     |                                    |
     |                                     | 5. Diffuser Mise à Jour d'État     |
     |<-------------------------------------------------------------------------|
     |                                     |                                    |
```

## Considérations de Sécurité

Le principal vecteur d'attaque dans l'orchestration multi-agents est la pollution de contexte malveillante via l'injection de prompt, où un agent compromis tente de propager des instructions hostiles à l'essaim.

Ce risque est fortement atténué par :

1. Une validation stricte du schéma (`"additionalProperties": false`) filtrant les données non structurées et neutralisant les charges utiles cachées.
2. Une non-répudiation cryptographique liant chaque action à la clé d'un agent spécifique, protégeant contre le ré-étiquetage de l'homme du milieu (man-in-the-middle).
3. Des moteurs de transition d'état déterministes agissant comme les seules entités capables de modifier le contexte partagé, isolant ainsi les modèles probabilistes.

## Rétrocompatibilité

Cette RFC étend le brouillon initial SDIC-1 sans rompre les implémentations existantes. Les configurations à agent unique peuvent ignorer en toute sécurité `correlation_id` et les actions de coordination d'essaim, maintenant la compatibilité avec le concept de base du Pilier 4 : Registre d'Actions décrit dans la spécification racine.

## Droit d'auteur

Droits d'auteur et droits connexes renoncés via CC0.
