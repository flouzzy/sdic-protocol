# ERC-1001 : Standard de Synchronisation du Registre d'Actions Multi-Agents

## Préambule

**ERC :** 1001
**Titre :** Standard de Synchronisation du Registre d'Actions Multi-Agents
**Auteur :** Charles EDOU NZE
**Type :** Standards Track
**Catégorie :** Extension SDIC-1
**Statut :** Brouillon
**Créé le :** 2026-08-18

## Résumé

Cette RFC propose une structure de données standardisée et un mécanisme de synchronisation pour le Registre d'Actions (Action Ledger) au sein de l'architecture SDIC-1, afin de prendre en charge le Consensus d'Essaim (Swarm Consensus) et l'orchestration multi-agents. Elle définit un schéma strict et déterministe pour les propositions d'intentions des agents et la vérification du consensus, afin d'empêcher les mutations d'état non autorisées dans des environnements partagés.

## Motivation

À mesure que les architectures natives de l'IA évoluent des intégrations mono-agent vers des essaims multi-agents, le risque de corruption de l'état se multiplie. Un hôte déterministe unique peut avoir besoin de réconcilier des intentions contradictoires provenant de plusieurs agents probabilistes. Sans un format d'entrée de registre standardisé et cryptographiquement vérifiable, conçu spécifiquement pour la synchronisation multi-agents, les agents pourraient écraser par inadvertance l'état de l'autre ou contourner les mécanismes de consensus. Ce standard impose l'intégrité structurelle et la non-répudiation cryptographique pour chaque étape d'un flux de travail multi-agents.

## L'Analogie de l'Orchestre Symphonique Sourd

Imaginez un orchestre symphonique de classe mondiale où chaque musicien est incroyablement talentueux mais totalement sourd. Ils peuvent parfaitement jouer de leur propre instrument, mais n'entendent pas ce que les autres jouent. S'ils commençaient tous à jouer en même temps en se basant uniquement sur leur propre partition, le résultat serait un bruit chaotique.

Pour créer une musique magnifique, ils s'en remettent à un chef d'orchestre central, à un métronome massif et très visible, ainsi qu'à un tableau d'affichage partagé. Un musicien ne se contente pas de jouer une note ; il écrit la note qu'il a l'intention de jouer sur une carte, la signe et la tend au chef d'orchestre. Le chef d'orchestre, qui peut voir les cartes de tout le monde, vérifie les signatures, contrôle si les notes s'intègrent dans la mesure actuelle de la symphonie, puis épingle officiellement les notes approuvées sur le tableau d'affichage partagé pour le temps suivant. Seules les notes figurant sur le tableau d'affichage sont réellement "jouées" dans la réalité.

**Correspondance Technique :**

- **Les Musiciens Sourds :** Les agents IA autonomes et probabilistes opérant de manière isolée (Sandbox).
- **La Note Prévue sur une Carte Signée :** La `MultiAgentActionLedgerEntry` (l'objet d'intention JSON), signée cryptographiquement par la clé privée de l'agent.
- **Le Chef d'Orchestre :** La Couche de Contrôle Déterministe de l'Application Hôte, responsable de la validation stricte (Déterminisme Sémantique).
- **Vérifier si les notes s'intègrent dans la mesure :** La fonction de validation du Consensus d'Essaim.
- **Le Tableau d'Affichage Partagé :** Le Registre d'Actions immuable (Action Ledger).
- **Jouer la Note :** Le Moteur de Transition d'État exécutant l'intention validée.

## Spécification

### 1. Le Schéma de l'Entrée du Registre d'Actions Multi-Agents

Toute proposition multi-agents soumise à l'application hôte DOIT se conformer au schéma JSON strict suivant. L'hôte DOIT rejeter toute charge utile qui échoue à la validation.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MultiAgentActionLedgerEntry",
  "type": "object",
  "properties": {
    "proposal_id": {
      "type": "string",
      "pattern": "^[0-9a-fA-F-]{36}$",
      "description": "UUIDv4 unique de la proposition."
    },
    "agent_id": {
      "type": "string",
      "description": "Identifiant public de l'agent proposant."
    },
    "swarm_id": {
      "type": "string",
      "description": "Identifiant du groupe de consensus."
    },
    "action_type": {
      "type": "string",
      "enum": ["propose_state_mutation", "vote_commit", "vote_reject"]
    },
    "payload": {
      "type": "object",
      "description": "La mutation d'état déterministe demandée.",
      "additionalProperties": false
    },
    "nonce": {
      "type": "integer",
      "description": "Entier strictement croissant pour empêcher les attaques par rejeu."
    },
    "signature": {
      "type": "string",
      "description": "Signature cryptographique de la proposition canonique."
    }
  },
  "required": [
    "proposal_id",
    "agent_id",
    "swarm_id",
    "action_type",
    "payload",
    "nonce",
    "signature"
  ],
  "additionalProperties": false
}
```

### 2. Représentation Canonique et Signature Cryptographique

Pour prévenir les attaques par réétiquetage (relabeling) ou par rejeu de la part d'acteurs malveillants modifiant la charge utile en transit, le champ `signature` DOIT valider une représentation canonique de l'objet.

1. **Canonisation :** L'hôte et l'agent DOIVENT générer une chaîne JSON déterministe stricte de l'entrée, en omettant complètement la clé `signature`. Les clés doivent être triées par ordre alphabétique, et les espaces blancs doivent être supprimés.
2. **Hachage :** La chaîne canonique est hachée à l'aide de SHA-256.
3. **Signature :** Le hachage est signé à l'aide de la Clé Privée de l'agent (ex. : Ed25519 ou ECDSA).

Soit $I$ l'objet d'Intention.
$Canonical(I)$ = chaîne JSON de $I$ excluant le champ `signature`, clés triées, sans espace blanc.
$\sigma = Sign_{AgentPrivKey}(Hash_{SHA256}(Canonical(I)))$

L'hôte DOIT vérifier cette signature par rapport au registre des agents approuvés pour le `swarm_id` donné avant d'admettre la proposition dans la fonction de Consensus d'Essaim.

## Justification

En appliquant `additionalProperties: false` de manière globale et en exigeant des signatures cryptographiques sur une charge utile canonisée excluant la signature elle-même, cette RFC élimine complètement la surface d'attaque pour les injections de clés non spécifiées et la falsification de la charge utile en transit dans les environnements multi-agents.

## Rétrocompatibilité

Il s'agit d'une nouvelle extension de la norme SDIC-1 (v1.0.0-draft) ciblant explicitement les configurations d'essaims multi-agents. Elle ne rompt pas les implémentations mono-agent existantes de l'Action Ledger, car elle introduit un nouveau schéma spécifique pour les propositions d'essaim plutôt que de modifier la table d'auditabilité temporelle de base.
