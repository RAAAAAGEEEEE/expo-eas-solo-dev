# Changelog

Format basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/). Ce projet suit [SemVer](https://semver.org/lang/fr/).

## [0.4.0] — 2026-07-27

- Documentation GitHub complète (README, LICENSE, CONTRIBUTING, SECURITY, docs/ARCHITECTURE/INSTALLATION/USAGE/CONFIGURATION/TROUBLESHOOTING/LIMITATIONS/PRIVACY_AND_SECURITY/LEGAL_AND_ATTRIBUTION) pour se conformer au standard de documentation du dépôt — absente jusqu'ici malgré la publication initiale.

## [0.3.0] — 2026-07-27

### Corrigé

- Le délai anti-vol iOS d'1h n'est pas universel : il ne se déclenche que si la Protection contre le vol d'appareil (Stolen Device Protection, iOS 17.3+) est activée et l'appareil hors lieu familier.
- Sévérité du statut "trader" DSA précisée : retrait automatique de la vente dans les 27 pays de l'UE depuis le 17/02/2025 pour une app déjà publiée, pas seulement un échec de build.

### Ajouté

- Quota EAS gratuit chiffré (15 builds Android + 15 iOS/mois, partagé entre tous les projets du compte).
- Précision sur la stabilité de `@expo/ui` (SwiftUI/Jetpack Compose stables depuis le SDK 56, mai 2026).
- Dates précises de dépréciation/suppression d'`expo-av` (déprécié SDK 54, supprimé SDK 55).
- Piste non validée pour les captures d'écran App Store sans Mac.
- Précision navigation App Store Connect pour Abonnements vs Achats intégrés.

## [0.2.0] — 2026-07-18

### Ajouté

- Section "Composants natifs avec `@expo/ui`" (SwiftUI Button API, convention `.ios.tsx`/`.android.tsx`, typage `SFSymbols`, limite de l'aperçu web).
- Section "RevenueCat en pratique" + `resources/revenuecat-api-v2.md` (cheatsheet API v2 : entitlements, offerings, produits, packages, clé Achat intégré, piège du `.p8` à téléchargement unique).
- Section "Aperçu live sur téléphone" (`expo start --tunnel`, contournement QR en environnement non interactif, piège de compatibilité SDK d'Expo Go).

## [0.1.0] — 2026-07-15

### Ajouté

- Version initiale du skill : posture générale, choix de stack, EAS/credentials/device pairing, conformité App Store/Play Store, i18n/accessibilité/design, tests, git.
- `resources/eas.json.template`, `resources/credentials-device-pairing-checklist.md`, `resources/appstore-submission-checklist.md`.
