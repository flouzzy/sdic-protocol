[English](SDIC-ERC-0005.md) | [Français](SDIC-ERC-0005.fr.md)

---

eip: 5
title: Le Cadre de Référence des 8 Étapes de la Maturité en Ingénierie de l'IA
author: SDIC-1 Architecture Group
type: Standards Track
category: ERC
status: Draft
created: 2026-06-09
---

## Résumé

Cet ERC définit les 8 Étapes de la Maturité en Ingénierie de l'IA, fournissant un cadre déterministe pour évaluer et faire évoluer les capacités en IA d'une organisation. Il standardise la progression d'une utilisation individuelle, probabiliste et non gérée vers des usines d'IA entièrement autonomes, auto-évaluées et hébergées sur une infrastructure partagée. Cette spécification introduit un modèle de maturité structurel conçu pour aligner le cycle de vie du développement avec une gouvernance stricte, des tests déterministes et une orchestration contextuelle.

## Motivation

À mesure que les organisations intègrent les Grands Modèles de Langage (LLM) et des agents autonomes dans leur cycle de vie de développement logiciel (SDLC), elles connaissent fréquemment une adoption très inégale. Tel que formulé à l'origine par Fabien Potencier dans ses réflexions sur la maturité en ingénierie de l'IA, une organisation n'existe pas à un stade unique de capacité ; elle présente plutôt une distribution de maturités à travers ses différentes équipes. Cette asymétrie crée des frictions d'intégration, des dépenses de calcul imprévisibles et des vulnérabilités de sécurité.

Établir une trajectoire standardisée et déterministe — allant d'une exécution locale isolée à une orchestration d'agents entièrement autonome au niveau de l'infrastructure — permet aux organisations de mettre à niveau systématiquement leurs flux de travail en ingénierie sans sacrifier la qualité ou le déterminisme.

## Analogie de Style Feynman : Le Réseau Municipal d'Eau

Considérez l'évolution de l'approvisionnement en eau d'une ville. Au départ, les habitants marchent jusqu'à une rivière et remplissent des seaux individuels à chaque fois qu'ils ont soif. Le processus est chaotique, non mesuré et dépend entièrement de l'effort individuel. Bientôt, les gens commencent à creuser leurs propres puits personnels et à construire des pompes ad hoc dans leurs jardins. Finalement, de petits quartiers mettent en commun leurs ressources pour construire des châteaux d'eau locaux, mais ceux-ci restent déconnectés du reste de la ville.

Reconnaissant l'inefficacité, une municipalité centrale intervient. Elle établit des normes universelles pour les dimensions des tuyaux, la pureté de l'eau et le comptage centralisé. Le flux de travail de la ville change : au lieu de chercher l'eau manuellement, les habitants s'appuient désormais sur des capteurs de pression déterministes et des vannes automatisées. Le système évolue vers un réseau interconnecté qui optimise le flux en fonction de contraintes mathématiques strictes. Les stations de pompage deviennent très avancées et s'auto-régulent, bien qu'elles nécessitent encore des superviseurs humains sur place. Enfin, l'ensemble du réseau atteint une autonomie totale : il détecte automatiquement les fuites, redirige la pression, planifie sa propre maintenance pendant les heures creuses et garantit la pureté sans aucune intervention humaine.

### Correspondance Technique

* **La Rivière / Les Seaux :** Les points d'accès IA non gérés (par exemple, les interfaces de chat standard) où les développeurs collent aveuglément du code sans contexte partagé ni configuration.
* **Les Puits Personnels :** Les développeurs individuels optimisant leurs environnements locaux avec des fichiers `AGENTS.md` personnels et des requêtes isolées.
* **Les Châteaux d'Eau Locaux :** Des équipes isolées partageant le contexte et les compétences, augmentant leur vélocité locale tout en s'écartant des normes organisationnelles.
* **Normes & Comptage :** La mise en œuvre de la gouvernance d'entreprise, SSO/SCIM, des budgets de jetons (tokens) et de la curation centrale des modèles et du contexte.
* **Capteurs de Pression & Vannes :** La refonte du processus imposant un développement "spécifications d'abord", des revues automatiques des demandes de tirage (PR) et des évaluations basées sur les risques.
* **Le Réseau Interconnecté :** Un système unifié où l'orchestration multi-agents, les serveurs de Protocole de Contexte de Modèle (MCP) partagés et le Développement Piloté par les Tests (TDD) agissent comme les dépendances fondamentales.
* **Stations Auto-régulées :** Des agents à haute autonomie fonctionnant localement sur les machines des développeurs, capables de générer et de livrer des unités de travail complètes sous la supervision d'un humain.
* **Le Réseau Autonome :** Des agents hébergés sur l'infrastructure fonctionnant dans des environnements isolés (sandbox) planifiés avec une traçabilité et une journalisation centralisées, exécutant de manière autonome des maintenances cycliques et des déploiements de fonctionnalités.

## Spécification

Le cadre définit 8 étapes séquentielles. La progression vers une étape ultérieure nécessite la réalisation des critères déterministes de l'étape précédente.

### Étape 1 : Le Vide

L'organisation n'a pas de stratégie de gouvernance définie. L'utilisation est caractérisée par l'absence de garde-fous, de fichiers de contexte partagés ou de clés API gérées. Les développeurs interagissent manuellement avec les modèles pour développer une intuition de leurs résultats probabilistes.

### Étape 2 : La Dérive

La divergence commence au niveau individuel. Des ingénieurs spécifiques mettent en œuvre des optimisations locales, utilisant des répertoires personnels de prompts, des compétences personnalisées et des fichiers `AGENTS.md` localisés pour automatiser leurs flux de travail. Cette étape représente la limite de l'inaction sans coût ; toute divergence supplémentaire introduit des risques structurels.

### Étape 3 : Les Îlots

L'écart d'outillage s'étend des individus aux équipes entières. Certaines équipes regroupent leurs configurations `AGENTS.md`, connectent des serveurs MCP et développent des compétences réutilisables, augmentant considérablement leur vélocité de production. Cela crée des monopoles d'efficacité localisés et des frictions structurelles avec les équipes aux pratiques plus anciennes.

### Étape 4 : Le Pari de la Standardisation

La direction impose le développement explicite de la capacité IA. L'ingénierie contextuelle est formalisée dans des répertoires centralisés. Une gouvernance obligatoire est introduite : intégration SSO/SCIM, analyse automatisée des secrets, contrôles de PR pour les résultats d'agents, et budgétisation unifiée des jetons.

### Étape 5 : La Refonte du Processus

Le paradigme d'exécution change. Le SDLC impose une architecture "spécifications d'abord" où la norme d'entrée est une spécification déterministe hébergée dans le dépôt. La revue de code passe d'une validation syntaxique à une évaluation basée sur les risques. Les barrières de l'Intégration Continue (CI) traitent les PR ouvertes par les agents de manière identique à celles ouvertes par les humains, exécutant des évaluations déterministes aux côtés des tests unitaires.

### Étape 6 : Le Système d'Exploitation

La composition de l'équipe se transforme. Les sprints incluent des emplacements de calcul pour les agents aux côtés des ingénieurs humains. Le TDD devient une exigence architecturale stricte pour empêcher les agents d'optimiser face à des contraintes faibles. Le contexte partagé (mémoire du projet, serveurs MCP) est géré comme une infrastructure centralisée.

### Étape 7 : L'Usine Intelligente

Les agents assemblent et livrent des unités de travail complètes et cohérentes avec une interaction humaine minimale. Cependant, l'exécution reste localisée. Les agents sont lancés à partir des environnements individuels des développeurs, exécutant des boucles locales et ouvrant des PR depuis des sandboxes localisées sans registre d'exécution partagé.

### Étape 8 : L'Usine Autonome

Les agents sont entièrement découplés du matériel local et déployés sur une infrastructure partagée. Les exécutions sont planifiées dans des sandboxes isolées disposant d'une traçabilité et d'une journalisation centralisées. Le système traite les opérations cycliques (mises à jour des dépendances, correctifs de sécurité) comme des tâches de daemon autonomes soumises à des critères de succès programmatiques. Les normes organisationnelles sont encodées directement dans les suites d'évaluation automatisées plutôt que dans les connaissances humaines.

## Structures de Données

Pour évaluer de manière déterministe la conformité à ce cadre, les systèmes DOIVENT implémenter le schéma JSON strict suivant pour les évaluations de maturité.

### Schéma MaturityAssessment

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MaturityAssessment",
  "type": "object",
  "properties": {
    "organization_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique de l'entité évaluée."
    },
    "current_stage": {
      "type": "integer",
      "minimum": 1,
      "maximum": 8,
      "description": "L'étape opérationnelle vérifiée actuelle."
    },
    "stage_configuration": {
      "type": "object",
      "properties": {
        "governance_enabled": {
          "type": "boolean",
          "description": "Vrai si la gouvernance d'entreprise et SCIM/SSO sont imposées."
        },
        "context_sharing_level": {
          "type": "string",
          "enum": ["NONE", "INDIVIDUAL", "TEAM", "ENTERPRISE"],
          "description": "La portée du partage de contexte et de compétences."
        },
        "infrastructure_hosting": {
          "type": "string",
          "enum": ["LOCAL", "SHARED_INFRASTRUCTURE"],
          "description": "L'environnement d'exécution des agents autonomes."
        }
      },
      "required": ["governance_enabled", "context_sharing_level", "infrastructure_hosting"],
      "additionalProperties": false
    }
  },
  "required": ["organization_id", "current_stage", "stage_configuration"],
  "additionalProperties": false
}
```

## Compatibilité Ascendante

Cette norme opère au niveau organisationnel et des processus, et est entièrement rétrocompatible avec la couche de contrôle déterministe et les frontières d'isolation cognitive définies dans `SDIC-RFC-0001` et le brouillon principal de SDIC-1. Aucun schéma existant ni environnement d'exécution ne nécessite de modification pour implémenter ce cadre.
