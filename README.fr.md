[English](README.md) | [Français](README.fr.md)

# SDIC-1: Systemic Deterministic Integration of AI Capabilities

- Author: Charles EDOU NZE

- Status: Draft

- Type: Standards Track

Created: 2026-05-25
---

## Abstract

Ce standard définit une architecture logicielle stricte qui intègre les capacités cognitives probabilistes (Grands Modèles de Langage, agents autonomes) dans des applications d'entreprise déterministes. L'objectif principal du SDIC-1 est d'éradiquer l'accès direct des agents autonomes aux couches de persistance et d'exécution en introduisant une frontière mathématique d'isolation, une non-répudiation cryptographique et un registre d'intentions standardisé.

## 1. Introduction

Pour comprendre le SDIC-1, imaginez une centrale nucléaire. L'IA (le Grand Modèle de Langage) est le cœur radioactif : il est immensément puissant, capable de générer des quantités massives d'énergie (l'intelligence), mais fondamentalement chaotique, probabiliste et dangereux s'il est exposé directement au monde extérieur.

L'application hôte traditionnelle est le bâtiment de confinement en béton et les barres de contrôle. Nous n'exposons jamais la ville directement au cœur radioactif. Au lieu de cela, nous captons la chaleur (l'**intention sémantique**), la canalisons à travers des tuyaux hautement résilients et strictement standardisés (le **Validateur de Schéma JSON**), et utilisons cette pression contrôlée pour faire tourner une turbine (l'**exécution déterministe**).

Le SDIC-1 est la spécification exacte pour construire ces tuyaux, garantissant qu'une force générative imprévisible puisse alimenter en toute sécurité des machines déterministes hautement réglementées.

## 2. Formalisme Mathématique de l'Isolation Cognitive

Le risque des architectures "AI-Native" réside dans la dégradation de l'état. Soit $S$ l'état déterministe du système hôte, et $C$ la capacité cognitive du modèle probabiliste.

Permettre à une IA de transiter directement l'état du système donne un résultat probabiliste :
$P(S_{t+1} | S_t, C) < 1$ (La transition d'état n'est pas déterministe).

Le SDIC-1 force une séparation stricte. Le modèle ne peut calculer qu'une Intention structurée $I$ basée sur un contexte filtré $X$ :
$I = C(X)$

Une fonction de validation déterministe $V(I)$ agit comme une porte binaire absolue. Soit $\Phi$ le schéma strict attendu :
$$
V(I) =
\begin{cases}
1, & \text{si } I \equiv \Phi \\
0, & \text{sinon}
\end{cases}
$$

La transition d'état est ensuite gérée uniquement par le moteur d'exécution déterministe $E$ :
$S_{t+1} = E(S_t, I)$ si et seulement si $V(I) = 1$.
Si $V(I) = 0$, l'état du système est préservé ($S_{t+1} = S_t$) et une exception est levée.

## 3. Spécification : La Pile Agnostique (Agnostic Stack)

Le protocole repose sur l'implémentation obligatoire de quatre couches agnostiques.

```text
+-------------------------------------------------------------+
|                      Application Hôte                       |
|   (Système Déterministe : Symfony, React, Go, etc.)         |
+------------------------------+------------------------------+
                               |
                   [1. Contexte Filtré & Prompt]
                               |
                               v
+-------------------------------------------------------------+
|                     IA / LLM Sandbox                        |
|   (Isolation Cognitive - Analyse & Émission d'Intention)    |
+------------------------------+------------------------------+
                               |
                    [2. Intention JSON brute]
                               |
                               v
+-------------------------------------------------------------+
|                Couche de Contrôle Déterministe              |
|   (Validation de Schéma Strict / Preuve Cryptographique)   |
+------------------------------+------------------------------+
                               |
                  [3. Intention Validée / Rejet]
                               |
                               v
+-------------------------------------------------------------+
|                        Action Ledger                        |
|   (Registre Immuable de Persistance et d'Exécution)        |
+-------------------------------------------------------------+
```

### 3.1. Isolation Cognitive (The Sandbox Boundary)

L'environnement d'exécution de l'IA doit être structurellement isolé.

- **Règle d'Or :** L'IA ne possède aucun droit d'écriture réseau, aucun accès direct à la base de données et aucun droit d'invoquer des API mutatives.
- **Contrat d'Interface :** L'IA agit exclusivement comme un transformateur sémantique. Elle émet un objet d'intention, incapable de l'exécuter elle-même.

### 3.2. Déterminisme Sémantique (Zero-Trust Schema Validation)

Toutes les sorties de la Sandbox doivent être traitées comme des entrées non fiables (`Untrusted Input`). La couche de validation doit appliquer une adhérence stricte au schéma.
Lors de la définition de Schémas JSON dans le dépôt, la rigueur doit être imposée en définissant `"additionalProperties": false` au niveau supérieur et sur les objets imbriqués applicables pour empêcher l'injection de clés non spécifiées.

**Exemple de Définition de Schéma Strict :**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TransferIntent",
  "type": "object",
  "properties": {
    "action": { "type": "string", "enum": ["transfer_funds"] },
    "target_id": { "type": "string", "pattern": "^[0-9a-fA-F-]{36}$" },
    "justification": { "type": "string" },
    "signature": { "type": "string" }
  },
  "required": ["action", "target_id", "justification", "signature"],
  "additionalProperties": false
}
```

### 3.3. Identité Cryptographique & Proof of Intent

Pour les opérations d'agents pleinement autonomes, une intention doit être non répudiable. Pour prévenir les attaques par rejeu et par réétiquetage (relabeling), l'Action Ledger requiert une signature cryptographique.
L'agent doit générer une signature $\sigma$ sur la représentation canonique de l'entrée complète (à l'exclusion du champ de signature lui-même).

$\sigma = Sign_{AgentPrivKey}(Hash(Canonical(I)))$

L'application hôte doit valider $\sigma$ par rapport à la Clé Publique connue de l'agent avant exécution.

### 3.4. L'Action Ledger (Auditabilité Temporelle)

Chaque action validée doit être enregistrée de manière immuable avant exécution.

| Champ | Type | Description |
|---|---|---|
| transaction_id | UUIDv4 | Identifiant unique et immuable de la transaction. |
| timestamp | DateTime | Marqueur temporel précis au niveau du système. |
| model_snapshot | String | Version exacte du modèle (ex. : claude-3-5-sonnet). |
| hyperparameters | Object | Configuration du modèle à l'exécution (Temperature, Top_P). |
| cognitive_context | Text | Copie exacte du prompt système et des données injectées. |
| raw_response | Text | Réponse brute inaltérée renvoyée par le modèle. |
| proof_of_intent | String | Signature cryptographique de l'intention canonique. |

### 3.5. Mémoire Continue & Consensus d'Essaim (Swarm Consensus)

Pour permettre la persistance de la conscience (rappelant les esprits numériques autonomes), l'IA ne peut pas stocker d'état en interne. L'état est maintenu dans l'Action Ledger et "réhydraté" de manière déterministe dans la fenêtre de contexte de l'IA à chaque invocation.

Pour les systèmes multi-agents (Swarm Consensus), les agents ne peuvent pas muter directement l'état partagé. Ils doivent émettre une $Intent_{Proposal}$ vers un registre déterministe partagé, ce qui déclenche une fonction de validation de consensus avant de valider l'état.

## 4. Considérations de Sécurité

- **Mitigation de l'Injection de Prompt :** Le Déterminisme Sémantique strict garantit que même si un modèle est détourné avec succès via une injection de prompt, toute sortie divergeant du Schéma JSON strict sera rejetée de manière déterministe par l'hôte.
- **Attaques par Réétiquetage (Relabeling Attacks) :** La validation de la signature sur la *représentation canonique* empêche les attaquants de modifier subtilement la charge utile en transit.
