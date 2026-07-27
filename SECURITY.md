# Sécurité

Ce dépôt contient uniquement des **instructions en Markdown** pour Claude Code — aucun code exécuté automatiquement, aucun service, aucune collecte de données. La surface de risque réelle concerne la façon dont le skill guide la manipulation de **secrets tiers** (tokens EAS, clés API RevenueCat, clés privées Apple `.p8`) pendant son usage. Détail complet de cette posture : [docs/PRIVACY_AND_SECURITY.md](docs/PRIVACY_AND_SECURITY.md).

## Signaler un problème

Si une instruction du skill recommande une pratique dangereuse (ex. exposition de secret, contournement de sécurité non justifié), ouvrir une issue publique sur ce dépôt — rien de sensible ne devrait transiter par ce canal puisque le dépôt lui-même ne contient jamais de secret.

## Ce que ce skill ne fait jamais

- Committer un secret (token, clé API, clé privée) dans un dépôt git.
- Demander à coller le contenu d'une clé privée dans le chat quand une lecture directe par chemin de fichier est possible.
- Contourner une protection anti-vol ou de sécurité Apple/Google sans l'accord explicite de l'utilisateur.
