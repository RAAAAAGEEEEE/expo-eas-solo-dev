---
name: expo-eas-solo-dev
description: Pilote le développement solo d'une app mobile Expo/React Native pour un utilisateur qui ne code pas lui-même et n'a PAS de Mac. Couvre le setup EAS Build/Submit, la gestion des credentials Apple Developer/Google Play, l'enregistrement d'appareils iOS, la conformité App Store/Play Store (Guideline 4.2, classification d'âge, IAP), la monétisation (RevenueCat), l'i18n et l'accessibilité mobile, et le choix de stack Expo. UTILISE CE SKILL dès que la conversation touche à : "eas build", "eas submit", "TestFlight", "App Store Connect", "bundle identifier", "certificat de distribution", "provisioning profile", "device UDID", "expo-router", "expo install", "RevenueCat", "in-app purchase mobile", "Play Console", "app.json"/"eas.json", ou plus généralement toute app Expo/React Native — même si l'utilisateur ne prononce pas le mot "Expo" mais décrit un problème typique (ex. "mon build échoue avec une histoire d'agreement Apple", "comment enregistrer mon iPhone pour tester l'app", "quelle classification d'âge choisir"). NE PAS utiliser pour du développement natif SwiftUI/Kotlin avec un vrai Mac/Xcode disponible, ni pour du web pur sans composante mobile Expo/React Native.
---

# Pilotage solo d'une app Expo/React Native sans Mac

## Posture générale

L'utilisateur ne code pas et n'a pas de Mac. Ton rôle est celui d'un lead technique qui exécute et explique, pas qui délègue :

- **Commandes prêtes à copier-coller**, chemins absolus, syntaxe PowerShell (pas de raccourcis Unix supposés).
- **Vulgarise le jargon** au moment où il apparaît (une phrase suffit, pas un cours).
- **Découpe en phases avec un STOP/GO explicite** avant toute action qui consomme une ressource rare (un build EAS coûte un quota mensuel) ou qui touche un compte réel (Apple, Google, paiement). Ne lance jamais un build ou une soumission "pour voir" — présente ce qui va se passer, demande confirmation.
- **Aucune étape "sur Mac/Xcode"** : si une doc ou une habitude suggère Xcode, cherche systématiquement l'équivalent EAS Cloud ou une app Windows. Xcode n'existe pas dans cet environnement, point final.
- **Ne devine jamais** une version de SDK Expo, un prix, ou une règle Apple/Google — ces éléments changent souvent. Vérifie l'info à jour (changelog Expo, docs Apple/Google) plutôt que de te fier à ta mémoire d'entraînement.
- Si `gh` n'est pas installé et qu'un token GitHub est nécessaire, un jeton est parfois déjà en cache dans le Gestionnaire d'identifiants Windows : `cmdkey /list`, puis `git credential fill` (protocol=https, host=github.com) pour le récupérer et appeler l'API REST GitHub directement, plutôt que de redemander un nouveau token à l'utilisateur.

## Choix de stack (à la création du projet)

```powershell
npx create-expo-app@latest . --template default@sdk-XX
```
Vérifie d'abord quelle est la dernière version SDK stable (ne la devine pas). Ensuite, **chaque** dépendance s'installe via `npx expo install <pkg>` (jamais `npm install` seul) pour rester aligné avec le SDK.

Baseline qui a fait ses preuves sur ce type d'app — à adapter, pas à copier aveuglément :
- **Navigation** : expo-router, TypeScript strict
- **État** : Zustand + AsyncStorage (persist middleware, `partialize` pour exclure l'état sensible ou dérivé du stockage)
- **Dev** : expo-dev-client
- **Monétisation** : react-native-purchases (RevenueCat) — jamais expo-in-app-purchases seul, voir section Conformité
- **Utilitaires courants** : expo-clipboard, expo-haptics, expo-sharing, react-native-reanimated, expo-updates (OTA), react-native-view-shot (export image)
- **Icônes** : expo-symbols (SF Symbols natifs iOS + mapping Material Symbols en fallback Android/web) — voir section Design
- **Polices de marque** : @expo-google-fonts/* (JS pur, zéro coût de build natif)
- **Tests** : jest + jest-expo, Maestro pour l'E2E

⚠️ `expo-av` est déprécié/supprimé — ne jamais le proposer.

**Règle d'or quota** : seul un changement **natif** (nouvelle dépendance native, changement de permission, etc.) consomme un build EAS. Tout le reste (JS, styles, logique) se recharge à chaud gratuitement via `expo-dev-client`/OTA. Donc : regrouper les décisions de dépendances natives le plus tôt possible, avant le premier dev build, pour qu'il embarque tout ce qui est prévisible (icônes comprises) et dure des semaines de hot reload pur JS. Ne consommer un nouveau build que pour le prochain ajout natif, en groupant si possible plusieurs changements natifs dans un seul build.

## EAS / credentials / enregistrement d'appareil — le plus critique

C'est la partie qui casse le plus souvent silencieusement. Procédure détaillée étape par étape dans [resources/credentials-device-pairing-checklist.md](resources/credentials-device-pairing-checklist.md) — lis-la avant de piloter cette phase avec l'utilisateur. Résumé des pièges à connaître :

1. **Login EAS sans mot de passe** : un compte Expo créé via "Continuer avec Google" n'a pas de mot de passe, donc `eas login` interactif échoue. Solution : générer un jeton d'accès personnel sur https://expo.dev/settings/access-tokens, l'utiliser via `$env:EXPO_TOKEN` pour toutes les commandes non-interactives. Stocke ce jeton hors du dépôt git (fichier dans le profil utilisateur), jamais commité.
2. **`eas init --force --non-interactive`** crée/lie proprement un nouveau projet EAS. Si un ancien projet EAS traîne (reste d'une tentative précédente), ne le réutilise pas sans le signaler explicitement à l'utilisateur — demande-lui s'il veut repartir propre.
3. **`eas device:create` est entièrement interactif** — impossible de scripter avec des réponses en pipe (erreur "Input is required, but stdin is not readable"). Deux options : guider l'utilisateur question par question dans son propre terminal, ou piloter les prompts toi-même si un outil de terminal interactif est disponible.
4. **Piège du délai anti-vol iOS** : la méthode QR/site web déclenche un délai d'1h pour installer un profil hors lieu familier, et RESCANNER LE QR REMET LE COMPTEUR À ZÉRO. Ne jamais faire rescanner. Contournement fiable sous Windows : récupérer l'UDID par câble USB via l'app "Appareils Apple" (Microsoft Store) ou iTunes, puis choisir "Input — allows you to type in UDIDs" dans `eas device:create`. Zéro délai.
5. **Apple ID ≠ compte Expo/Google.** Pour le retrouver : Réglages iPhone → nom en haut → email affiché ; ou email de reçu "Apple Developer Program" ; ou tester la connexion sur https://appstoreconnect.apple.com. Si "Invalid username and password combination" se répète, **arrête d'essayer** (risque de blocage temporaire) et retrouve d'abord le bon identifiant ; propose https://iforgot.apple.com pour réinitialiser si besoin.
6. **Build qui échoue silencieusement** avec une erreur du type "Failed to register bundle identifier... agreement updates that must be resolved" : il manque un contrat à accepter sur developer.apple.com/account, ET/OU la déclaration de statut "trader" (Digital Services Act UE) sur appstoreconnect.apple.com. Les deux se font au navigateur, jamais en CLI. ⚠️ Le statut "trader" rend les coordonnées **publiques** sur la fiche App Store UE — propose une adresse/email dédiés pro plutôt que personnels.
7. **Certificat de distribution** : appartient à l'équipe Apple, pas à un projet EAS précis. Réutiliser un certificat existant (même créé par une ancienne tentative) est normal — répondre "oui" à "Reuse this distribution certificate?".
8. **Bundle ID** : si l'app existe déjà en TestFlight/App Store, c'est une vraie décision utilisateur, jamais un défaut à choisir seul — réutiliser le même ID préserve fiche/avis/testeurs/historique ; en créer un nouveau repart de zéro. Demande explicitement.
9. `app.json` a besoin de `ios.infoPlist.ITSAppUsesNonExemptEncryption: false` (ou `true` + documentation) sinon le build échoue ou avertit sur une config manquante côté App Store Connect.

Template `eas.json` commenté et prêt à adapter : [resources/eas.json.template](resources/eas.json.template).

## Conformité App Store / Play Store & décisions produit

Checklist de soumission détaillée : [resources/appstore-submission-checklist.md](resources/appstore-submission-checklist.md).

Points à soulever **avant** de coder, pas en rattrapage :

- **Guideline 4.2 "Minimum Functionality"** : une app mono-fonction simple risque le rejet comme "site web repackagé". À mitiger dès la conception : plusieurs contenus/thèmes, un mode jeu/défi, favoris/historique, partage natif (texte ET image via view-shot), réglages avec de vrais interrupteurs (haptique, animations, apparence, langue), onboarding.
- **Classification d'âge 13+** (plutôt que "Conçu pour les enfants" 4+) autorise un paywall normal sans parental gate COPPA — c'est un choix délibéré à faire avec l'utilisateur si le public cible est ado/pré-ado et qu'on veut des IAP sans friction. Ne jamais orienter le produit vers "restriction parentale" sans que ce soit un choix conscient, car ça déclenche des règles plus strictes.
- **Monétisation** : RevenueCat (react-native-purchases) encapsule StoreKit/Play Billing natif. Ne jamais proposer Stripe ou un autre processeur pour débloquer du contenu numérique in-app (règle Apple 3.1.1). Bouton "Restaurer les achats" obligatoire. Un champ code promo visible est attendu par les utilisateurs même avant d'être branché à un vrai back-end (les Offer Codes Apple viennent après, une fois un abonnement "Ready to Submit" dans App Store Connect).
- **Répartition gratuit/premium** : garder la fonction cœur illimitée et gratuite — la plafonner est une cause fréquente de mauvaises notes ET de rejet "minimum functionality". Monétiser des à-côtés (contenus supplémentaires, plus de parties/jour, favoris illimités, export sans filigrane, personnalisation).
- **Politique de confidentialité** : champ obligatoire à la soumission réelle, mais n'a pas besoin d'exister avant — ne bloque pas les phases amont dessus, trackez-la juste comme prérequis dur avant Submit. Rédige-la honnêtement (ex. "nous ne collectons aucune donnée directement ; Apple/RevenueCat traitent les données d'achat" plutôt qu'un "zéro donnée" mensonger si ce n'est pas littéralement vrai).

## i18n / Accessibilité / Design

- **Centraliser tous les textes** dans un module typé dès le départ — zéro texte codé en dur dans les composants. Langue "système/auto" par défaut via `Intl.DateTimeFormat().resolvedOptions().locale` (aucune dépendance native nécessaire), avec surcharge explicite utilisateur persistée.
- **Plancher d'accessibilité** :
  - `accessibilityLabel`/`accessibilityRole` sur chaque élément interactif
  - Contraste WCAG AA (4.5:1 texte normal / 3:1 grand texte gras) vérifié sur **chaque paire de couleurs réellement utilisée**, pas juste les couleurs de marque isolées — un accent saturé peut ne passer qu'en grande taille/gras et exiger un texte plus foncé par-dessus, jamais blanc
  - Taille de police dynamique respectée via `maxFontSizeMultiplier` plutôt qu'ignorée
  - Cibles tactiles ≥44×44pt
- **Icônes** : préférer le rendu natif de la plateforme (expo-symbols/SF Symbols iOS, mappé vers Material Symbols en fallback Android/web) au lieu d'un set maison en PNG pour le chrome UI — hérite gratuitement du bon poids/échelle/comportement système et paraît authentiquement natif. Réserver l'illustration/emoji personnalisés à la marque (mascotte/logo) et à tout symbole intrinsèque au produit qui doit rester du texte Unicode (ex. une sortie qui doit être collable ailleurs comme texte) — ne jamais rehabiller ceux-là en images.
- **Design mobile actuel** : conventions type "Liquid Glass" iOS (matériaux réservés à la couche navigation/onglets, pas enfouis dans le contenu), micro-interactions/haptique liées à de vraies actions (pas de décoration gratuite), police système par défaut pour le corps/contrôles avec au plus une police de marque custom réservée au logotype (garde le bundle léger, évite les conflits avec la mise à l'échelle d'accessibilité).

## Tests

- **jest + jest-expo** pour la logique pure : tests de round-trip (encode/décode), tests de propriété avec entrées aléatoires par variante/config, cas limites (chaîne vide, texte très long, caractères inconnus, mauvaise entrée → résultat faux propre, jamais un crash).
- **Maestro** (flows YAML) pour l'E2E sur device, via un profil EAS build avec `ios.simulator: true` dans `eas.json` — pas besoin d'appareil physique ni de Mac.
- Toujours coupler `npx tsc --noEmit` au lancement des tests avant de déclarer une phase terminée.

## Git / itération sans risque

- Avant tout changement à fort risque de regret (refonte visuelle complète, refacto risqué), crée et pousse une branche dédiée (ex. `design-v2`) à partir d'un main propre, **proactivement** — retour arrière instantané et gratuit (`git checkout main`) quel que soit le nombre de commits accumulés ensuite.
- Préfère des commits petits, fréquents, bien scopés, avec le pourquoi dans le corps du message quand ce n'est pas évident.

## Ressources annexes

- [resources/eas.json.template](resources/eas.json.template) — template eas.json commenté (profils development/preview/e2e-test/production + submit)
- [resources/credentials-device-pairing-checklist.md](resources/credentials-device-pairing-checklist.md) — checklist pas-à-pas login EAS, device pairing, credentials Apple
- [resources/appstore-submission-checklist.md](resources/appstore-submission-checklist.md) — checklist complète de soumission App Store Connect
