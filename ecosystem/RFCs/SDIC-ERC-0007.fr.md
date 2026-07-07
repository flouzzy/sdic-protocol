---
eip: 7
title: Validation de Schéma JSON en Continu pour la Réduction de la Latence des Garde-Fous
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
category: Core
created: 2026-07-07
---

[English](SDIC-ERC-0007.md) | [Français](SDIC-ERC-0007.fr.md)

## Résumé

Cette norme propose une extension architecturale au protocole SDIC-1 pour atténuer la latence de la couche des Garde-Fous (Guardrails) en introduisant la **Validation de Schéma JSON en Continu** (ou "Fast-Path Validation"). Au lieu d'attendre la génération complète d'une intention par un Grand Modèle de Langage (LLM) avant d'appliquer une validation déterministe, cette spécification définit un mécanisme pour intercepter et valider les tokens en temps réel, interrompant le processus de génération prématurément si la sortie s'écarte du schéma attendu.

## Motivation

Dans l'architecture actuelle de SDIC-1, la Couche de Contrôle Déterministe applique une validation stricte du Schéma JSON sur l'intention sémantique brute émise par le Bac à Sable IA. Bien que cela garantisse un déterminisme sémantique, le processus de validation ne se produit qu'une fois que le modèle a terminé de générer l'intention entière. Dans les scénarios impliquant des chemins de raisonnement très complexes ou de grandes structures JSON, la génération de tokens invalides consomme des ressources de calcul inutiles et ajoute une latence importante avant que l'application hôte puisse rejeter l'intention.

En introduisant un mécanisme de validation en continu, nous pouvons évaluer de manière déterministe la structure générée token par token et interrompre de manière proactive les générations non conformes, réduisant drastiquement la latence et le gaspillage de calcul dans les pipelines d'IA d'entreprise.

## L'Analogie du Rayon X des Douanes

Imaginez un poste frontalier international où des camions transportant des marchandises entrent dans un pays. Traditionnellement, le processus douanier fonctionne comme un point d'inspection final : un camion roule jusqu'à un entrepôt sécurisé, décharge l'intégralité de sa cargaison, et une équipe d'inspecteurs vérifie manuellement chaque boîte par rapport au manifeste officiel. Si une seule boîte est interdite ou manquante, toute la cargaison est rejetée, rechargée et renvoyée. Ce processus est sécurisé mais incroyablement lent et inefficace, surtout si l'article interdit a été chargé tout à l'avant du camion.

Pour améliorer l'efficacité sans compromettre la sécurité, nous installons un tunnel à rayons X à grande vitesse à l'entrée même de la frontière. Alors que le camion traverse lentement le tunnel, le rayon X scanne la cargaison couche par couche en temps réel. Le système de numérisation est connecté à un ordinateur déterministe comparant la section transversale de la cargaison aux dimensions mathématiques des articles autorisés sur le manifeste. Si le rayon X détecte une anomalie — une forme qui ne correspond pas au schéma strict — le tunnel clignote immédiatement en rouge, arrête le camion sur place et l'empêche d'entrer plus loin dans le pays.

Dans notre architecture SDIC-1 :

- **Le Camion et la Cargaison** représentent le flux de tokens générés par le Grand Modèle de Langage.
- **Le Point d'Inspection Final** représente la couche traditionnelle des Garde-Fous, qui attend la charge utile JSON complète avant la validation.
- **Le Tunnel à Rayons X à Grande Vitesse** représente le mécanisme de Validation de Schéma JSON en Continu, analysant la sortie de manière incrémentale.
- **Les Dimensions Mathématiques** incarnent le Schéma JSON strict appliqué avec `"additionalProperties": false`.
- **L'Arrêt du Camion** représente l'interruption proactive de la génération de tokens dès que le modèle s'écarte du schéma.

## Spécification

### 1. Architecture de Validation Fast-Path

Le processus de validation en continu intercepte le flux de sortie de l'IA au sein de l'application hôte déterministe, avant l'étape finale de validation du Schéma JSON.

```text
+-----------------------+
|   Bac à Sable IA      |
+-----------+-----------+
            | (Flux de Tokens)
            v
+-----------------------+
| Validateur en Continu | --- [Anomalie Détectée] ---> (Annuler Génération)
|  (Couche Fast-Path)   |
+-----------+-----------+
            | (Flux de Tokens Validé)
            v
+-----------------------+
| Contrôle Déterministe |
|   (Schéma JSON)       |
+-----------------------+
```

### 2. Schéma de Configuration du Validateur en Continu

La politique de validation en continu DOIT adhérer au schéma JSON suivant, appliquant mathématiquement les limites du filtre fast-path.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "StreamingValidatorConfig",
  "type": "object",
  "properties": {
    "validator_id": {
      "type": "string",
      "format": "uuid"
    },
    "target_schema_ref": {
      "type": "string",
      "description": "Référence au Schéma JSON strict (ex: ActionLedgerEntry) auquel le flux doit adhérer."
    },
    "buffer_size_bytes": {
      "type": "integer",
      "minimum": 1
    },
    "abort_on_structural_deviation": {
      "type": "boolean"
    },
    "max_allowed_latency_ms": {
      "type": "integer",
      "minimum": 0
    }
  },
  "required": [
    "validator_id",
    "target_schema_ref",
    "buffer_size_bytes",
    "abort_on_structural_deviation"
  ],
  "additionalProperties": false
}
```

## Considérations de Sécurité

La mise en œuvre de la validation en continu introduit un risque mineur de fuite de données partielles si l'application hôte persiste accidentellement ou agit sur des fragments JSON incomplets avant que la génération ne soit interrompue. L'application hôte DOIT s'assurer que le validateur en continu agit uniquement comme un filtre en temps réel, et qu'aucune transition d'état ou opération de persistance ne se produit tant que la Couche de Contrôle Déterministe n'a pas validé avec succès l'intention JSON finalisée et complète.

## Compatibilité Ascendante

Ce RFC est entièrement rétrocompatible. Les systèmes implémentant la version `v1.0.0-draft` de SDIC-1 peuvent déployer le Validateur en Continu comme un proxy en amont de la Couche de Contrôle Déterministe. Si la validation en continu n'est pas disponible ou échoue, le système se rabat naturellement sur la validation finale et complète du Schéma JSON.
