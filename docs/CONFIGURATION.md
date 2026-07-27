# Configuration

Ce skill n'a **aucun fichier de configuration propre** — pas de `.env`, pas de paramètres à régler avant usage. Il se comporte uniquement comme un ensemble d'instructions chargées par Claude Code.

## Ce qui ressemble à de la configuration mais n'en est pas

- **`EXPO_TOKEN`**, les clés API RevenueCat (`sk_...`), les clés `.p8` Apple : ce sont des secrets **du projet que le skill aide à construire**, jamais du skill lui-même. Ils ne doivent jamais être committés dans ce dépôt ni dans aucun dépôt — voir [PRIVACY_AND_SECURITY.md](PRIVACY_AND_SECURITY.md).
- **`resources/eas.json.template`** : un modèle à copier et adapter dans le projet cible, pas un fichier que ce skill lit lui-même.

## Personnaliser le skill pour son propre usage

Toute adaptation se fait en éditant directement les fichiers Markdown (`SKILL.md`, `resources/*.md`) — pas de mécanisme de surcharge ou de variables d'environnement prévu. Voir [../CONTRIBUTING.md](../CONTRIBUTING.md) pour la marche à suivre.
