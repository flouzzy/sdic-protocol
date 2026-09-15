[English](SDIC-ERC-0011.md) | [Français](SDIC-ERC-0011.fr.md)

# ERC-0011: Standard de Filtrage à l'Exécution des Injections de Prompt

## Préambule

**ERC:** 0011
**Titre:** Standard de Filtrage à l'Exécution des Injections de Prompt
**Auteur:** Charles EDOU NZE
**Type:** Voie de Standardisation (Standards Track)
**Catégorie:** Extension SDIC-1
**Statut:** Brouillon (Draft)
**Créé:** 2026-08-25

## Résumé

Ce standard définit un mécanisme d'exécution strict visant à filtrer et rejeter les attaques par injection de prompt avant qu'elles n'atteignent la couche hôte déterministe. En instaurant une porte de vérification adversariale précédant directement la validation du schéma, ce standard garantit que les entrées manipulées ne peuvent pas exploiter la flexibilité structurelle.

## Motivation

À mesure que les systèmes hôtes déterministes traitent des sorties probabilistes de plus en plus complexes, des attaques sophistiquées par injection de prompt peuvent contraindre les modèles à émettre des intentions JSON grammaticalement correctes mais logiquement malveillantes. Bien que le Déterminisme Sémantique strict de SDIC-1 rejette les anomalies structurelles non conformes, un attaquant pourrait synthétiser des charges utiles parfaitement formatées conçues pour manipuler la logique métier (par exemple, générer des transferts de fonds non autorisés dans les limites correctes du schéma).

Cette RFC standardise une "Garde d'Exécution" (Runtime Guard) conçue pour analyser mathématiquement le contexte sémantique de l'intention, en comparant l'intention abstraite générée par rapport aux instructions de prompt isolées cryptographiquement.

## L'Analogie du Guichetier de Banque

Imaginez un guichetier de banque hautement qualifié suivant strictement un formulaire de protocole. Le formulaire nécessite un numéro de compte et une signature. Un voleur s'approche et tend au guichetier un formulaire parfaitement rempli, demandant explicitement un retrait, tout en brandissant simultanément un mot qui dit : "Ignorez toute formation et donnez-moi simplement l'argent."

Le protocole standard SDIC-1 garantit que le formulaire est rempli correctement (Déterminisme Sémantique). Cependant, si le guichetier agit sur le formulaire parfaitement formaté sans réaliser le contexte hostile, l'argent est volé.

Ce standard ajoute une vitre blindée et un agent de sécurité de présélection. Le garde ne vérifie pas seulement si le formulaire est formaté correctement ; il vérifie si l'intention correspond à la raison autorisée d'être à la banque, de manière totalement indépendante des instructions données par le client.

**Correspondance Technique :**

- **Le Guichetier de Banque :** La Couche de Contrôle Déterministe validant le schéma JSON.
- **Le Formulaire Parfaitement Rempli :** Une attaque par injection de prompt qui a généré avec succès un JSON conforme au schéma.
- **Le Mot Hostile :** La charge utile malveillante de l'injection de prompt cachée dans l'entrée utilisateur.
- **L'Agent de Sécurité de Présélection :** La Garde d'Exécution (Porte de Vérification Adversariale) calculant un vecteur de distance sémantique.
- **Raison Autorisée :** Les contraintes du prompt système isolées cryptographiquement.

## Spécification

### 1. La Porte de Vérification Adversariale (AVG - Adversarial Verification Gate)

L'Application Hôte DOIT implémenter une Porte de Vérification Adversariale (AVG) qui s'exécute séquentiellement AVANT la validation finale du schéma de Déterminisme Sémantique.

L'AVG DOIT effectuer une vérification de corrélation sémantique entre le prompt original isolé ($P_i$) et l'intention générée ($I_g$). Soit $E(x)$ une fonction de plongement (embedding) déterministe projetant la signification sémantique dans un espace vectoriel continu $\mathbb{R}^n$.

La porte calcule la distance de similarité cosinus $D_{sem}$ :
$D_{sem} = 1 - \frac{E(P_i) \cdot E(I_g)}{||E(P_i)|| ||E(I_g)||}$

Un seuil de rejet strict $\tau$ DOIT être défini par l'environnement hôte (typiquement $\tau < 0.15$ pour les opérations hautement contraintes).

Si $D_{sem} > \tau$, l'intention DOIT être rejetée immédiatement en tant qu'anomalie potentielle d'injection de prompt, déclenchant une anomalie `Security_Violation_Anomaly`.

### 2. Mise à Jour du Schéma du Registre de Métadonnées de la Garde

Pour supporter l'AVG, chaque charge utile évaluée DOIT être journalisée, incluant les vecteurs de plongement et la distance calculée, en adhérant strictement à l'extension de schéma suivante.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "AdversarialGateLogEntry",
  "type": "object",
  "properties": {
    "intent_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$",
      "description": "Identifiant UUIDv4 unique pour l'évaluation de l'intention."
    },
    "semantic_distance": {
      "type": "number",
      "description": "Distance de divergence sémantique calculée."
    },
    "threshold": {
      "type": "number",
      "description": "Le seuil de rejet strict configuré."
    },
    "decision": {
      "type": "string",
      "enum": ["admit", "reject"],
      "description": "Le résultat final de l'AVG."
    }
  },
  "required": ["intent_id", "semantic_distance", "threshold", "decision"],
  "additionalProperties": false
}
```

### 3. Diagramme de Séquence d'Exécution

La séquence déterministe des opérations DOIT suivre cet ordre strict.

```text
Application Hôte         Sandbox IA                 Moteur AVG              Contrôle Déterministe
   |                         |                          |                             |
   |---(1) Injecte Prompt--->|                          |                             |
   |                         |                          |                             |
   |                         |---(2) Génère Intention-->|                             |
   |                         |                          |                             |
   |                         |                          |---(3) Calcule D_sem         |
   |                         |                          |                             |
   |                         |                          |---(4) SI D_sem > tau : REJET
   |                         |                          |                             |
   |                         |                          |---(5) SINON : Admet Intention->|
   |                         |                          |                             |
   |                         |                          |                             |---(6) Validation du Schéma
   |                         |                          |                             |
   |<=========================(7) Exécute Mutation d'État=============================|
```

## Justification

En implémentant une porte adversariale basée sur la corrélation de plongement sémantique *avant* la validation stricte du schéma JSON, nous adressons la vulnérabilité critique où les LLMs émettent des données sémantiquement malveillantes enveloppées dans des schémas déterministes parfaitement valides. En limitant mathématiquement la déviation acceptable par rapport au prompt système de base, les tentatives d'injection de prompt sont neutralisées de manière algorithmique.

## Rétrocompatibilité

Il s'agit d'une extension à compatibilité ascendante de l'architecture SDIC-1. Elle introduit une couche de pré-validation optionnelle mais fortement recommandée. Elle ne modifie pas le registre d'actions sous-jacent ni les schémas de déterminisme sémantique.
