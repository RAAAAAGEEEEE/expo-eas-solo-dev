# Confidentialité et sécurité

## Ce que ce skill est

Un ensemble de fichiers Markdown chargés en contexte par Claude Code. Il n'exécute rien par lui-même, ne fait aucun appel réseau, ne stocke aucune donnée. Toute action réelle (appel API, commande shell, lecture de fichier) est effectuée par Claude Code au moment de l'usage, pas par ce dépôt.

## Secrets manipulés pendant l'usage (pas stockés dans ce dépôt)

En suivant ce skill sur un vrai projet, Claude Code sera amené à manipuler, pour le compte de l'utilisateur :
- un **token d'accès EAS** (`EXPO_TOKEN`) ;
- une **clé secrète API RevenueCat** (`sk_...`) ;
- une **clé privée Apple `.p8`** (clé "Achat intégré" StoreKit 2).

Le skill instruit explicitement de :
- ne **jamais committer** un secret dans un dépôt git ;
- stocker un token hors du dépôt du projet (fichier du profil utilisateur) ;
- **lire un fichier `.p8` directement depuis son chemin sur disque** plutôt que de demander à l'utilisateur de coller son contenu dans le chat, pour éviter qu'un secret ne traîne inutilement dans l'historique de conversation ;
- traiter toute clé secrète comme un vrai secret même quand elle est reçue en clair de l'utilisateur pour un usage ponctuel (ex. appel `curl`).

## Contournements de sécurité — posture

Ce skill documente un contournement légitime (récupération d'UDID par câble plutôt que par QR code) pour éviter un délai de sécurité Apple **dans un cas d'usage normal** (développeur enregistrant son propre appareil sur son propre projet). Il ne documente et ne documentera jamais de contournement visant à échapper à un contrôle de sécurité sans le consentement du propriétaire de l'appareil/compte concerné.

## Aucune collecte

Ni ce dépôt ni son usage via Claude Code ne collectent de données sur l'utilisateur ou sur les projets construits avec son aide.
