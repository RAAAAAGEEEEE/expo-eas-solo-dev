# expo-eas-solo-dev

Skill Claude Code pour piloter le développement d'une app mobile **Expo/React Native** quand l'utilisateur **ne code pas lui-même et n'a pas de Mac**.

## Problème résolu

Construire et publier une app iOS/Android en solo, sans savoir coder et sans Mac, oblige à trouver l'équivalent EAS Cloud/Windows de chaque étape habituellement documentée "sur Xcode" — et à éviter une longue liste de pièges silencieux (credentials Apple, quota de builds, conformité App Store) qui ne pardonnent pas à un non-développeur livré à lui-même. Ce skill encode ces pièges et la posture de pilotage nécessaire, tirés d'une session de développement réelle de bout en bout (app iOS Mysterymoji, Expo SDK 57).

## Public cible

Un utilisateur solo, non-développeur, qui délègue l'exécution technique à Claude Code sur Windows (ou tout OS sans Xcode), et qui veut publier sur l'App Store et/ou le Play Store.

## Statut

**Beta.** Construit et affiné sur un seul projet réel (Mysterymoji, iOS, Expo SDK 57), complété par des audits documentaires avec vérification des sources officielles — voir [docs/LIMITATIONS.md](docs/LIMITATIONS.md) pour ce qui est documenté mais pas encore traversé en conditions réelles (notamment le cycle de publication Android complet).

## En quoi il diffère du skill officiel `expo/skills`

L'équipe Expo publie des skills officiels ([github.com/expo/skills](https://github.com/expo/skills), MIT, plugin Claude Code `expo@claude-plugins-official`). Au 2026-09-29, leur liste couvre le framework (Expo Router, animations, `@expo/ui`, mises à jour de SDK, modules natifs) et les services EAS (`eas-app-stores`, `eas-update`, `eas-workflows`...). Ils expliquent comment utiliser Expo et EAS correctement. Ils ne sont pas écrits pour une personne précise.

Ce dépôt part de cette personne : quelqu'un qui **ne code pas** et **n'a pas de Mac**. La différence tient à trois choses.

| | `expo/skills` (officiel) | `expo-eas-solo-dev` |
| --- | --- | --- |
| Public visé | Agent qui assiste un développeur | Agent qui pilote à la place d'un non-développeur |
| Environnement | Non spécifique | Windows sans Mac : commandes PowerShell, jamais d'étape « sur Xcode », équivalent EAS Cloud à chaque fois |
| Posture | Référence technique | STOP/GO avant toute action qui consomme un quota de build ou touche un compte réel |
| Apple / Google | Build, soumission, TestFlight, métadonnées (`eas-app-stores`) | Pièges de compte et de calendrier : contrats Apple, statut trader DSA, appareil iOS sans Mac, règle Google Play des 12 testeurs, Small Business Program, classification d'âge, rejets |
| Monétisation | Pas de skill dédié dans la liste vérifiée | RevenueCat (SDK et API v2), règle 3.1.1, restauration des achats |

Les deux sont complémentaires : gardez l'officiel pour la référence technique à jour d'Expo et d'EAS, et celui-ci pour la conduite de projet d'un non-développeur sans Mac. En cas de contradiction sur un fait daté (version de SDK, quota, règle de store), la documentation Expo, Apple ou Google fait foi, et ce skill demande de la revérifier plutôt que de se fier à sa propre valeur écrite.

Ce dépôt n'est pas affilié à Expo.

## Prérequis

- [Claude Code](https://claude.com/claude-code) installé.
- Windows (le skill donne des commandes PowerShell et des solutions spécifiquement pensées pour l'absence de Mac — utilisable sur macOS/Linux mais les commandes devront être adaptées).
- Aucun compte développeur Apple/Google requis pour installer le skill lui-même — seulement pour l'utiliser sur un vrai projet.

## Installation

Voir [docs/INSTALLATION.md](docs/INSTALLATION.md) pour le détail. En résumé :

```powershell
git clone https://github.com/RAAAAAGEEEEE/expo-eas-solo-dev.git "$env:USERPROFILE\.claude\skills\expo-eas-solo-dev"
```

Le skill est alors disponible dans **toutes** les sessions Claude Code futures, sur tous les projets.

## Utilisation

Rien à invoquer manuellement — le skill se déclenche automatiquement dès qu'une conversation touche à EAS, TestFlight, App Store Connect, RevenueCat, `@expo/ui`, ou plus largement une app Expo/React Native. Détail et exemples : [docs/USAGE.md](docs/USAGE.md).

## Ce que couvre le skill

- Décisions à impact calendrier/revenus à prendre au jour 1 (règle Google Play des 12 testeurs, Small Business Program à 15 %, classification d'âge)
- Choix de stack Expo (dépendances recommandées, `expo-av` à éviter, etc.)
- Setup EAS Build/Submit et credentials Apple Developer/Google Play sans Mac
- Enregistrement d'un appareil iOS sans passer par le piège du délai anti-vol
- Conformité App Store/Play Store (Guideline 4.2, classification d'âge, IAP)
- Monétisation via RevenueCat (SDK + API v2)
- Composants natifs `@expo/ui` (SwiftUI/Jetpack Compose)
- i18n, accessibilité, tests (Jest, Maestro)

Détail architecture : [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Sécurité et confidentialité

Ce skill manipule des instructions sur des secrets réels (tokens EAS, clés API RevenueCat, clés privées `.p8` Apple). Il ne collecte, n'envoie ni ne stocke rien lui-même — voir [docs/PRIVACY_AND_SECURITY.md](docs/PRIVACY_AND_SECURITY.md) pour la posture exacte.

## Limites honnêtes

- Écrit et validé sur **un seul projet réel** (iOS) — pas encore éprouvé sur un cycle Android complet de bout en bout.
- Cinq cas d'évaluation dans [`evals/evals.json`](evals/evals.json), à passer à la main ou avec l'outil d'évals de skill-creator ; aucun harnais automatique ne les exécute encore (voir [docs/LIMITATIONS.md](docs/LIMITATIONS.md)).
- Les prix, quotas et règles Apple/Google cités sont datés de leur dernière vérification (voir CHANGELOG) — le skill lui-même consigne qu'il faut revérifier, pas se fier à la valeur écrite indéfiniment.

## Roadmap (non contractuelle)

- Retour d'expérience Android vécu de bout en bout (le contenu Play Console/signing est documenté mais pas encore éprouvé sur un cycle complet).
- Ajout d'exemples de manifestes i18n/accessibilité complets.
- Validation en conditions réelles de la procédure de recomposition des captures d'écran App Store.

## Contribuer

Voir [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

[MIT](LICENSE).

## Documentation détaillée

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- [docs/INSTALLATION.md](docs/INSTALLATION.md)
- [docs/USAGE.md](docs/USAGE.md)
- [docs/CONFIGURATION.md](docs/CONFIGURATION.md)
- [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)
- [docs/LIMITATIONS.md](docs/LIMITATIONS.md)
- [docs/PRIVACY_AND_SECURITY.md](docs/PRIVACY_AND_SECURITY.md)
- [docs/LEGAL_AND_ATTRIBUTION.md](docs/LEGAL_AND_ATTRIBUTION.md)
