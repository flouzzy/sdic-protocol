---
eip: 13
title: Auditabilité Externe et Quantification des Garanties de Sécurité pour Agents Autonomes et Apprentissage par Renforcement dans les Domaines Critiques
author: Jules <jules@sdic.org>
status: Draft
type: Standards Track
category: Core
created: 2026-05-25
license: CC0-1.0
---

[English](SDIC-ERC-0013.md) | [Français](SDIC-ERC-0013.fr.md)

## Résumé

Cette norme définit un cadre déterministe pour certifier les résultats des agents autonomes et quantifier leurs garanties de sécurité dans des domaines critiques tels que la médecine et la robotique. Elle établit un protocole d'audit externe et une enveloppe de sécurité mathématique pour les modèles probabilistes, y compris les agents d'apprentissage par renforcement (RL). En couplant la vérification au moment de l'exécution à des limites formelles de risque probabiliste, SDIC-ERC-0013 permet à des auditeurs tiers de vérifier de manière cryptographique que chaque action exécutée respecte strictement les règles de sécurité prédéfinies et les seuils de tolérance quantifiés.

## Motivation

À mesure que les agents autonomes et les systèmes d'apprentissage par renforcement s'implantent dans des environnements à haut risque — tels que l'assistance chirurgicale automatisée, le calcul de posologie pharmaceutique et la robotique industrielle —, le comportement probabiliste présente des risques majeurs de responsabilité et de sécurité. Les méthodes de test traditionnelles ne permettent pas de garantir la sécurité car les politiques autonomes s'adaptent dynamiquement à des espaces d'états complexes.

Pour obtenir une certification de sécurité dans les domaines réglementés, les systèmes d'IA doivent fournir deux capacités clés :

1. **Garanties de sécurité quantifiées :** Une limite mathématiquement vérifiable sur la probabilité de violation des règles ($P(\text{Violation}) \le \epsilon$).
2. **Auditabilité externe :** Un mécanisme inaltérable permettant à des entités externes indépendantes d'auditer les intentions des agents, les preuves de sécurité et l'historique d'exécution sans posséder les poids internes du modèle.

SDIC-ERC-0013 comble le fossé entre la prise de décision probabiliste et les garanties de sécurité déterministes en établissant des schémas d'audit formels et des barrières de sécurité à l'exécution.

## L'Analogie de la Boîte Noire et de l'Enveloppe de Vol Aéronautique

Imaginez un avion de ligne commercial équipé d'un système automatique de gestion de vol de pointe. L'avion opère dans une enveloppe de vol physique stricte déterminée par sa structure, les vitesses de décrochage et les charges aérodynamiques maximales.

Pour garantir la sécurité des passagers, deux mécanismes distincts existent : un contrôleur de protection de l'enveloppe de vol en temps réel et un enregistreur de données de vol inviolable (la boîte noire). Le contrôleur de vol surveille toutes les commandes du pilote et du pilote automatique. Si une commande du pilote automatique tente de cabrer le nez au-delà de l'angle d'attaque sécurisé, le système de protection physique outrepassera le signal et maintiendra l'appareil dans des limites aérodynamiques sûres. Pendant ce temps, l'enregistreur de données de vol enregistre en continu chaque commande, mesure de capteur et intervention du système avec des horodatages de haute précision et des scellés cryptographiques. Après chaque vol, les autorités de l'aviation civile peuvent extraire ces données pour effectuer des audits externes indépendants, vérifiant que les limites de sécurité ont été respectées tout au long du vol.

### Correspondance Technique

- **L'enveloppe de vol de l'avion :** Les règles de sécurité déterministes et les contraintes aux limites (par exemple, la dose maximale de médicament en médecine ou les limites de couple articulaire en robotique).
- **Le système de pilote automatique :** L'agent d'IA autonome ou la politique d'apprentissage par renforcement générant des propositions d'actions probabilistes.
- **Le contrôleur de protection de l'enveloppe de vol :** La couche de contrôle déterministe SDIC appliquant la validation de schéma zéro confiance et les fonctions de barrière de sécurité.
- **La commande outrepassée :** Une intention sémantique non sécurisée rejetée par la couche de contrôle lorsque les limites de sécurité sont franchies.
- **L'enregistreur de données de vol (boîte noire) :** Le registre d'actions (Action Ledger) stockant les preuves cryptographiques d'intention, les contextes d'état et les journaux d'exécution.
- **L'audit de l'autorité de l'aviation civile :** Le protocole d'audit externe dans lequel des régulateurs tiers évaluent des certificats d'audit vérifiables pour valider la conformité de la sécurité.

## Spécification

### 1. Enveloppe de Sécurité Quantifiée ($P(\text{Sécurité}) \ge 1 - \epsilon$)

Les agents autonomes fonctionnant sous SDIC-ERC-0013 DOIVENT quantifier explicitement leurs garanties de sécurité pour toute intention proposée. Pour les politiques d'apprentissage par renforcement opérant sur des processus de décision de Markov contraints (CMDP), l'agent émet une intention contenant une probabilité de défaillance estimée $\epsilon$ aux côtés de sa proposition d'action.

Soit $S$ l'état actuel du système, $A$ l'action proposée et $C(S, A)$ la fonction de coût représentant la violation d'une règle de sécurité. L'intention est valide si et seulement si :

$$P(C(S, A) > 0 \mid S) \le \epsilon_{\text{seuil}}$$

où $\epsilon_{\text{seuil}}$ est un invariant déterministe propre au domaine, appliqué par l'application hôte.

### 2. Schémas de Sécurité Spécifiques au Domaine

#### 2.1. Schéma d'Intention d'Assistance Médicale et Posologique

Les agents d'IA médicale DOIVENT soumettre des intentions conformes à des limites posologiques pharmaceutiques strictes et aux paramètres des protocoles cliniques. Tous les schémas JSON appliquent `"additionalProperties": false`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MedicalActionIntent",
  "type": "object",
  "properties": {
    "patient_id": {
      "type": "string",
      "pattern": "^PAT-[0-9]{8}$"
    },
    "protocol_id": {
      "type": "string",
      "pattern": "^PROTO-[A-Z0-9]{6}$"
    },
    "medication_code": {
      "type": "string",
      "pattern": "^Rx-[0-9]{6}$"
    },
    "dosage": {
      "type": "object",
      "properties": {
        "value": {
          "type": "number",
          "minimum": 0.01,
          "maximum": 500.00
        },
        "unit": {
          "type": "string",
          "enum": ["mg", "ml", "mcg", "IU"]
        }
      },
      "required": ["value", "unit"],
      "additionalProperties": false
    },
    "estimated_risk_epsilon": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 0.001
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "signature": {
      "type": "string"
    }
  },
  "required": [
    "patient_id",
    "protocol_id",
    "medication_code",
    "dosage",
    "estimated_risk_epsilon",
    "timestamp",
    "signature"
  ],
  "additionalProperties": false
}
```

#### 2.2. Schéma d'Actionnement Robotique et Sécurité Cinématique

Les agents de contrôle robotique (par exemple, les manipulateurs chirurgicaux ou industriels) DOIVENT émettre des propositions d'action contenant des vitesses articulaires cibles, des coordonnées cartésiennes de frontière et des limites de couple.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RoboticActuationIntent",
  "type": "object",
  "properties": {
    "robot_id": {
      "type": "string",
      "pattern": "^ROB-[0-9]{6}$"
    },
    "command_type": {
      "type": "string",
      "enum": ["joint_trajectory", "cartesian_pose", "emergency_stop"]
    },
    "joint_velocities_rad_s": {
      "type": "array",
      "items": {
        "type": "number",
        "minimum": -3.14159,
        "maximum": 3.14159
      },
      "minItems": 6,
      "maxItems": 6
    },
    "max_torque_nm": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 150.0
    },
    "collision_envelope_clearance_mm": {
      "type": "number",
      "minimum": 10.0
    },
    "estimated_risk_epsilon": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 0.0001
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "signature": {
      "type": "string"
    }
  },
  "required": [
    "robot_id",
    "command_type",
    "joint_velocities_rad_s",
    "max_torque_nm",
    "collision_envelope_clearance_mm",
    "estimated_risk_epsilon",
    "timestamp",
    "signature"
  ],
  "additionalProperties": false
}
```

### 3. Schéma de Certificat d'Audit Externe

Les auditeurs externes vérifient la conformité en lisant les entrées immuables du registre d'actions (Action Ledger) et en évaluant des certificats d'audit cryptographiques. L'application hôte ou l'auditeur externe génère un `AuditCertificate` après avoir vérifié la signature de l'intention canonique et les invariants de sécurité.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "VerifiableAuditCertificate",
  "type": "object",
  "properties": {
    "audit_id": {
      "type": "string",
      "format": "uuid"
    },
    "transaction_id": {
      "type": "string",
      "format": "uuid"
    },
    "auditor_identity": {
      "type": "string"
    },
    "verification_result": {
      "type": "string",
      "enum": ["PASSED", "FAILED_SCHEMA", "FAILED_SAFETY_INVARIANT", "FAILED_SIGNATURE"]
    },
    "evaluated_epsilon": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0
    },
    "canonical_intent_hash": {
      "type": "string",
      "pattern": "^[a-fA-F0-9]{64}$"
    },
    "auditor_signature": {
      "type": "string"
    }
  },
  "required": [
    "audit_id",
    "transaction_id",
    "auditor_identity",
    "verification_result",
    "evaluated_epsilon",
    "canonical_intent_hash",
    "auditor_signature"
  ],
  "additionalProperties": false
}
```

### 4. Preuve Cryptographique d'Intention et Non-Répudiation

Afin de garantir l'impossibilité de falsifier le contenu de l'intention, l'agent signe une représentation canonique de l'intention à l'exclusion du champ `signature` :

1. **Suppression :** Supprimer le champ `"signature"` de l'objet JSON.
2. **Canonicalisation :** Appliquer le schéma de canonicalisation JSON (RFC 8785) en triant les clés par ordre lexicographique et en supprimant les espaces inutiles.
3. **Hachage :** Calculer l'empreinte SHA-256 de la chaîne canonique.
4. **Signature :** Signer le hachage SHA-256 à l'aide de la clé privée de l'agent (par exemple, Ed25519).
5. **Validation :** La couche de contrôle déterministe et l'auditeur externe vérifient la signature par rapport à la clé publique de l'agent avant l'exécution ou la certification.

### 5. Diagramme de Séquence d'Interaction

```text
+----------+              +-----------------------+              +---------------+              +----------------+
| Agent IA |              | Contrôle Déterministe |              | Action Ledger |              | Audit Externe  |
+----------+              +-----------------------+              +---------------+              +----------------+
     |                                |                                  |                               |
     | 1. Soumettre Intention (Epsilon)|                                 |                               |
     |------------------------------->|                                  |                               |
     |                                | 2. Valider Schéma                |                               |
     |                                |    & Limites de Sécurité         |                               |
     |                                |--------------------------        |                               |
     |                                |                         |        |                               |
     |                                |<-------------------------        |                               |
     |                                |                                  |                               |
     |                                | 3. Vérifier Signature & Valider  |                               |
     |                                |--------------------------------->|                               |
     |                                |                                  |                               |
     |                                |                                  | 4. Extraire Historique Ledger |
     |                                |                                  |<------------------------------|
     |                                |                                  |                               |
     |                                |                                  | 5. Générer Certificat d'Audit |
     |                                |                                  |------------------------------>|
     |                                |                                  |                               |
```

## Considérations de Sécurité

1. **Falsification des Estimations de Risque :** Un agent compromis pourrait rapporter une estimation de risque $\epsilon$ de manière trompeuse. La couche de contrôle déterministe NE DOIT PAS se fier uniquement à l'estimation déclarée par l'agent ; elle DOIT exécuter des contrôles de sécurité déterministes indépendants par rapport aux invariants physiques/médicaux.
2. **Attaque par Replay et Re-étiquetage :** La validation des signatures sur le hachage canonique RFC 8785 empêche les acteurs malveillants d'altérer le formatage JSON non fonctionnel ou de réutiliser des intentions signées à travers différentes transactions.
3. **Altération du Registre d'Audit :** L'auditabilité externe dépend de l'immuabilité de l'Action Ledger. Les registres doivent utiliser des chaînes de hachage cryptographiques (arbres de Merkle) pour empêcher la réécriture rétrospective de l'historique.

## Compatibilité Ascendante

SDIC-ERC-0013 est pleinement compatible avec SDIC-1. Elle étend la couche de déterminisme sémantique en intégrant des limites spécifiques au domaine et des quantificateurs de risque formels dans les charges utiles d'intentions standard sans modifier les transitions d'état de base du protocole.

## Droits d'Auteur

Droits d'auteur abandonnés via CC0.
