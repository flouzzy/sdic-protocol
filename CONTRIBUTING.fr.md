[English](CONTRIBUTING.md) | [Français](CONTRIBUTING.fr.md)

# Contribuer à SDIC-1

Nous accueillons avec plaisir les contributions à la norme SDIC-1 (Systemic Deterministic Integration of AI Capabilities). Ce dépôt est entièrement constitué de spécifications au format Markdown. Afin de garantir la plus haute qualité de la norme, nous appliquons des directives de contribution strictes.

## Directives Générales

- **Documentation Bilingue :** La documentation du projet doit être bilingue, fournissant des versions en français et en anglais avec des liens de navigation croisée en haut de chaque fichier. Si vous ajoutez ou mettez à jour un fichier, vous devez fournir sa contrepartie traduite.
- **Pas de Charabia d'Entreprise :** Tous les fichiers Markdown doivent être parfaitement formatés, syntaxiquement corrects et exempts de charabia d'entreprise (corporate fluff) ou de texte de remplacement.
- **Analogies de Style Feynman :** Lors de la rédaction ou de l'expansion de la documentation standard, incluez des analogies de style Feynman pour fournir une clarté et une élégance techniques profondes.
- **Exemples Concrets :** Tout exemple concret, application ou algorithme est le bienvenu en parallèle des explications pour maintenir la profondeur technique de la norme.

## Spécifications Techniques

- **Agnostique au Langage :** Les spécifications techniques doivent être agnostiques au langage de programmation et hautement techniques.
- **Validations Déterministes :** Utilisez des exemples stricts de Schémas JSON ou des diagrammes de séquence UML ASCII le cas échéant.
- **Schéma JSON Strict :** Lors de la définition de Schémas JSON au sein du dépôt, imposez la rigueur en définissant `"additionalProperties": false` au niveau supérieur (et sur les objets imbriqués applicables) pour empêcher l'ajout de clés non spécifiées.

## Demande de Commentaires (RFCs)

- **Format Protocole Internet Standard :** Les propositions de RFC doivent suivre le format classique de Protocole Internet (IETF RFC).
- **Répertoire :** Les RFC doivent être stockées dans le répertoire `ecosystem/RFCs/`.

## Directives pour les Commits

- **Commits Conventionnels :** Les commits doivent strictement utiliser le format des [Commits Conventionnels (Conventional Commits)](https://www.conventionalcommits.org/).

## Directives pour les Pull Requests (Demandes de Tirage)

Les descriptions des Pull Requests doivent strictement respecter la disposition suivante contenant ces en-têtes exacts :

1. **Summary of Changes** (Résumé des Changements)
2. **Rationale & Architectural Impact** (Justification & Impact Architectural)
3. **Proposed Specification Changes** (Modifications de Spécification Proposées)
4. **Backward Compatibility** (Compatibilité Ascendante)

## Formatage

- Le formatage Markdown est vérifié et corrigé à l'aide de `markdownlint-cli`.
- Nous utilisons une configuration `.markdownlint.json` assouplie (désactivant les règles telles que MD013 pour la longueur de ligne et MD041 pour les en-têtes de première ligne). Assurez-vous que vos modifications sont conformes à la configuration avant de les soumettre.
