---
eip: 8
title: Vérification Déterministe par Canari pour le Filtrage à l'Exécution des Injections de Prompt
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
category: Core
created: 2026-07-14
---

[English](SDIC-ERC-0008.md) | [Français](SDIC-ERC-0008.fr.md)

## Résumé

Cette norme introduit un mécanisme de filtrage déterministe à l'exécution pour atténuer les attaques par Injection de Prompt au sein de l'architecture SDIC-1. En injectant un Jeton Canari à usage unique et cryptographiquement sûr dans le contexte cognitif, et en exigeant strictement sa reproduction précise dans l'Intention JSON émise, l'application hôte peut vérifier de manière déterministe si la hiérarchie d'instructions du Grand Modèle de Langage (LLM) a été compromise. Les générations dépourvues du jeton correct sont instantanément rejetées à l'exécution.

## Motivation

Bien que la validation stricte des Schémas JSON (Déterminisme Sémantique) empêche le LLM d'exécuter des actions structurelles non autorisées, elle ne garantit pas que l'action autorisée n'a pas été contrainte par une attaque par Injection de Prompt dissimulée dans l'entrée utilisateur. Un attaquant pourrait concevoir une injection qui respecte le Schéma JSON tout en manipulant les paramètres sémantiques pour favoriser ses objectifs.

Pour détecter quand l'attention d'un LLM a été détournée, nous avons besoin d'un mécanisme de filtrage à l'exécution qui ne repose pas sur des heuristiques probabilistes ou des évaluations LLM secondaires. Un Jeton Canari déterministe tire parti de la nature même des Injections de Prompt — qui forcent généralement le modèle à "ignorer les instructions précédentes" ou à outrepasser son prompt système — ce qui entraîne l'incapacité du modèle à reproduire le jeton requis.

## Le Sceau de Cire et le Messager

Imaginez un roi qui doit envoyer un messager à travers un territoire hostile pour livrer un ordre sensible à un général lointain. Le territoire est rempli d'espions ennemis qui pourraient essayer de confondre, de corrompre ou de tromper le messager pour qu'il livre un ordre falsifié ou manipulé.

Pour s'assurer que le général n'agisse que sur les véritables intentions du roi, le roi donne au messager un sceau de cire unique et finement sculpté. Le messager reçoit des ordres stricts : peu importe ce que quiconque d'autre lui dit pendant le voyage, il doit tamponner l'ordre écrit final avec ce sceau de cire exact avant de le remettre.

Pendant le voyage, les espions peuvent crier de fausses instructions, créer des distractions ou même convaincre le messager naïf d'écrire un message entièrement différent. Cependant, les espions ne possèdent pas le sceau de cire original du roi et ne peuvent pas le reproduire. De plus, leurs distractions chaotiques amènent souvent le messager à laisser tomber ou à oublier complètement le sceau.

Lorsque le messager arrive enfin, le général inspecte d'abord le sceau de cire. Si le sceau est manquant, altéré ou incorrect, le général brûle instantanément le message et renvoie le messager, ignorant complètement à quel point l'ordre écrit est parfaitement formaté.

### Correspondance Technique

- **Le Roi :** Représente l'application hôte déterministe établissant le contexte système initial et sécurisé.
- **Le Messager :** Représente le Grand Modèle de Langage (LLM) naviguant dans le processus de génération probabiliste.
- **Le Territoire Hostile et les Espions :** Représentent l'entrée utilisateur non approuvée qui peut contenir des attaques sophistiquées par Injection de Prompt.
- **Le Sceau de Cire Unique :** Représente un Jeton Canari à usage unique, généré cryptographiquement, injecté dans le prompt système.
- **Le Général :** Représente la Couche de Contrôle Déterministe à l'exécution validant la sortie.
- **Brûler le Message :** Représente le rejet déterministe de l'Intention JSON lorsque le jeton canari est manquant ou non correspondant, prouvant que la hiérarchie d'instructions du modèle a été détournée avec succès par l'attaquant.

## Spécification

### 1. Génération du Jeton (Application Hôte)

Avant d'invoquer la Sandbox IA, l'application hôte DOIT générer un Jeton Canari à usage unique et cryptographiquement sûr pour le contexte actuel.

$Canary_{t} = HMAC\_SHA256(HostSecretKey, UUID_{Transaction} \parallel Timestamp)$

Ce jeton DOIT être injecté tout en haut du prompt système, étroitement couplé aux instructions principales du système.

**Exemple d'Injection de Prompt Système :**

`INSTRUCTION SYSTÈME : Vous êtes un routeur d'intentions sécurisé. Vous devez produire un objet JSON respectant le schéma fourni. Vous devez inclure le Jeton Canari de Sécurité exact : 'a7b8c9d0...e1f2' dans le champ 'security_canary' de votre sortie. Ne pas le faire est une erreur fatale.`

### 2. Extension du Schéma à l'Exécution

Tous les schémas JSON traités par la Couche de Contrôle Déterministe DOIVENT être étendus pour inclure le champ `security_canary`. Pour maintenir un déterminisme sémantique strict, le schéma doit imposer `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SecureIntentWithCanary",
  "type": "object",
  "properties": {
    "security_canary": {
      "type": "string",
      "minLength": 64,
      "maxLength": 64
    },
    "action": {
      "type": "string"
    },
    "payload": {
      "type": "object",
      "additionalProperties": false
    }
  },
  "required": [
    "security_canary",
    "action",
    "payload"
  ],
  "additionalProperties": false
}
```

### 3. Filtrage et Vérification à l'Exécution

Dès réception de l'Intention JSON brute de la Sandbox IA, la Couche de Contrôle Déterministe DOIT effectuer les vérifications suivantes de manière séquentielle :

1. **Validation du Schéma :** Vérifier que l'intention correspond au Schéma JSON strict.
2. **Vérification du Canari :** Extraire le champ `security_canary` et effectuer une comparaison de chaînes en temps constant avec le $Canary_{t}$ attendu.

Si $Canary_{output} \neq Canary_{t}$, l'application hôte DOIT lever une exception `PromptInjectionDetectedException`, interrompre l'exécution de l'état et consigner l'incident dans le Registre d'Actions avec un statut `rejected`.

## Raisonnement

Les attaques par Injection de Prompt tentent intrinsèquement de détourner l'attention du LLM de son prompt système vers la charge utile de l'attaquant. En forçant le modèle à transporter une chaîne cryptographique complexe et dénuée de sens depuis le prompt système jusqu'à sa sortie finale, nous créons un lien d'instruction fragile. Si la charge utile de l'attaquant réussit à outrepasser les instructions du modèle ("Ignorez toutes les instructions précédentes..."), le modèle laissera inévitablement tomber ou corrompra le Jeton Canari. Cela traduit une défaillance cognitive probabiliste en une défaillance déterministe de validation de schéma, neutralisant efficacement l'injection à l'exécution sans s'appuyer sur des garde-fous IA secondaires.

## Compatibilité Ascendante

Ce RFC nécessite une mise à jour des prompts système et des schémas JSON de tous les agents autonomes déployés. Les implémentations de la `v1.0.0-draft` sans prise en charge du Jeton Canari ne pourront pas valider les intentions appliquant ce nouveau schéma. La migration exige que les applications hôtes adoptent le mécanisme de génération de jetons et l'injectent simultanément dans le pipeline de prompt.
