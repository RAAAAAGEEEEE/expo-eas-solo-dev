---
name: expo-eas-solo-dev
description: Pilote le développement solo d'une app mobile Expo/React Native pour un utilisateur qui ne code pas lui-même et n'a PAS de Mac. Couvre le setup EAS Build/Submit, la gestion des credentials Apple Developer/Google Play, l'enregistrement d'appareils iOS, la conformité App Store/Play Store (Guideline 4.2, classification d'âge, IAP), la monétisation (RevenueCat), l'i18n et l'accessibilité mobile, et le choix de stack Expo. UTILISE CE SKILL dès que la conversation touche à : "eas build", "eas submit", "TestFlight", "App Store Connect", "bundle identifier", "certificat de distribution", "provisioning profile", "device UDID", "expo-router", "expo install", "@expo/ui", "boutons natifs Apple/Android", "SwiftUI"/"Jetpack Compose" côté React Native, "RevenueCat", "in-app purchase mobile", "Play Console", "app.json"/"eas.json", ou plus généralement toute app Expo/React Native — même si l'utilisateur ne prononce pas le mot "Expo" mais décrit un problème typique (ex. "mon build échoue avec une histoire d'agreement Apple", "comment enregistrer mon iPhone pour tester l'app", "quelle classification d'âge choisir"). NE PAS utiliser pour du développement natif SwiftUI/Kotlin avec un vrai Mac/Xcode disponible, ni pour du web pur sans composante mobile Expo/React Native.
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
- **UI native (boutons, contrôles)** : `@expo/ui` (SwiftUI iOS / Jetpack Compose Android) pour de vrais composants système plutôt qu'une imitation stylée en JS — voir section dédiée
- **Polices de marque** : @expo-google-fonts/* (JS pur, zéro coût de build natif)
- **Tests** : jest + jest-expo, Maestro pour l'E2E

⚠️ `expo-av` est déprécié/supprimé — ne jamais le proposer.

**Règle d'or quota** : seul un changement **natif** (nouvelle dépendance native, changement de permission, etc.) consomme un build EAS. Tout le reste (JS, styles, logique) se recharge à chaud gratuitement via `expo-dev-client`/OTA. Donc : regrouper les décisions de dépendances natives le plus tôt possible, avant le premier dev build, pour qu'il embarque tout ce qui est prévisible (icônes comprises) et dure des semaines de hot reload pur JS. Ne consommer un nouveau build que pour le prochain ajout natif, en groupant si possible plusieurs changements natifs dans un seul build.

## Aperçu live sur téléphone — à proposer très tôt (dès les premiers écrans qui tournent)

Dès qu'il y a un premier écran affichable (souvent avant même la fin du scaffold), propose à l'utilisateur de voir l'app tourner en direct sur son téléphone, sans attendre une phase dédiée. Zéro coût de build EAS, zéro compte requis :

```powershell
cd apps/mobile   # ou la racine du projet Expo
npx expo install --check   # aligne les versions avant de lancer
npx expo start --tunnel
```

- `--tunnel` est **indispensable** si la machine de dev n'est pas sur le même réseau Wi-Fi que le téléphone (quasi toujours vrai dans un environnement cloud/sandbox) — sinon `--lan` suffit.
- Si `@expo/ngrok` n'est pas déjà présent, l'installer en devDependency **local au projet** (`npx expo install --dev @expo/ngrok` ou `pnpm add -D @expo/ngrok`) — une install globale seule ne suffit pas toujours à satisfaire la détection d'Expo CLI.
- En environnement non interactif (`CI=1` ou terminal piloté par un agent), le QR code ne s'affiche pas toujours proprement. Récupère l'URL du tunnel directement via l'API locale de ngrok : `curl http://localhost:4040/api/tunnels` → champ `public_url`. Construis le lien `exp://<host-sans-protocole>` à partir de ça.
- Donne ce lien `exp://...` à l'utilisateur avec l'instruction : ouvrir l'app **Expo Go**, puis "Enter URL manually" (pas besoin de scanner un QR si le lien est déjà en main).

⚠️ **Piège de compatibilité SDK à vérifier avant de promettre que ça marche** : Expo Go publié sur l'App Store iOS a régulièrement un train de retard sur la dernière version SDK à cause des délais de revue Apple (ex. bloqué sur SDK 54 alors qu'un projet neuf est en SDK 57+). Ne suppose jamais que la version installée sur le téléphone de l'utilisateur supporte le SDK du projet — vérifie l'état actuel (changelog expo.dev/changelog) et préviens l'utilisateur que si Expo Go refuse avec une erreur de version, les options sont : (a) mettre à jour Expo Go depuis l'App Store si une version plus récente est sortie entre-temps, (b) `eas go` (build TestFlight personnalisé d'Expo Go — nécessite un compte Apple Developer payant, donc un vrai STOP/GO avec l'utilisateur avant de le lancer), (c) tester sur simulateur iOS via Expo CLI en attendant. Ne jamais lancer `eas go` sans confirmation explicite, exactement comme un build EAS classique.

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

## RevenueCat en pratique — API v2 et pièges App Store Connect

Au-delà du SDK client (`react-native-purchases`), RevenueCat expose une **API REST v2** qui permet de configurer tout le dashboard (entitlements, offerings, produits) sans passer par leur interface — utile si l'utilisateur préfère confier une clé plutôt que cliquer lui-même dans un dashboard tiers. Cheatsheet complète avec exemples curl : [resources/revenuecat-api-v2.md](resources/revenuecat-api-v2.md).

Points essentiels, ceux qui coûtent le plus de temps s'ils sont découverts en cours de route :

- **Deux clés RevenueCat différentes.** La clé **publique** SDK (`appl_...`/`goog_...`, une par plateforme, embarquée dans l'app comme `EXPO_PUBLIC_REVENUECAT_API_KEY_IOS`/`_ANDROID`) — sans risque à demander directement à l'utilisateur. La clé **secrète** API (`sk_...`) pilote tout le dashboard par API — à traiter comme un vrai secret (jamais dans le code commité, jamais réaffichée en clair après usage), mais recevable en direct de l'utilisateur pour l'utiliser dans des appels `curl`/API le temps de la session.
- **Deux clés App Store Connect différentes, faciles à confondre.** La clé générale "**API App Store Connect**" (Utilisateurs et accès → Intégrations → onglet "API App Store Connect" — celle qu'utilise par ex. `eas submit`, rôle App Manager/Admin, sert pour piloter builds/métadonnées/TestFlight). La clé dédiée "**Achat intégré**" (In-App Purchase Key, onglet séparé "Achat intégré" sur la même page — StoreKit 2, sert uniquement à ce que RevenueCat valide les reçus d'achat). Une clé existante déjà utilisée pour autre chose (ex. celle d'EAS Submit) **ne peut pas être réutilisée** pour RevenueCat si son fichier `.p8` n'a pas été conservé — il en faut une nouvelle, dédiée.
- ⚠️ **Le fichier `.p8` ne se télécharge qu'une seule fois**, quelle que soit la clé (générale OU Achat intégré) — Apple ne permet aucun re-téléchargement ultérieur. Si l'utilisateur ne l'a pas sauvegardé au moment de la génération, la clé est perdue pour de bon, il faut en régénérer une autre. **Insister lourdement sur ce point avant qu'il clique sur "Générer", pas après coup.**
- Une fois le `.p8` sur le disque de l'utilisateur, **le lire directement depuis son chemin de fichier** (accès disque local disponible dans ce type d'environnement) plutôt que de lui demander de coller le contenu de la clé privée dans le chat — plus propre, évite qu'un secret traîne inutilement dans l'historique de conversation.
- Abonnements (mensuel/annuel) et achat unique (à vie) vivent dans **deux sections séparées** d'App Store Connect, toutes deux sous l'onglet **"Fonctionnalités"** de la fiche app (le libellé exact et l'emplacement bougent régulièrement dans l'UI Apple — vérifie l'intitulé du moment plutôt que de le supposer) : "**Abonnements**" impose de créer d'abord un **groupe d'abonnements** (le regroupement qui gère l'upgrade/downgrade entre plans), puis chaque abonnement individuel à l'intérieur ; "**Achats intégrés**" liste directement les produits (non consommable pour un déblocage à vie, consommable pour un crédit rechargeable). Un utilisateur non-technique confond facilement les deux car les deux écrans se ressemblent (même formulaire prix/nom/description) — nomme explicitement dans quelle section tu le fais naviguer à chaque étape, ne dis jamais juste "va dans les achats".
- Avant de créer quoi que ce soit via l'API, **toujours lister l'état existant** (`GET /v2/projects/{id}/apps`, `/entitlements`, `/offerings`, `/products`) — un projet RevenueCat peut déjà contenir une configuration placeholder (bundle ID bidon type `com.vibecode.*`, produits de démo) issue d'un onboarding automatique ou d'un essai antérieur avec un autre outil. Ne jamais assumer qu'un projet est vide ou cohérent sans vérifier, et signaler toute incohérence trouvée avant de la corriger.

## i18n / Accessibilité / Design

- **Centraliser tous les textes** dans un module typé dès le départ — zéro texte codé en dur dans les composants. Langue "système/auto" par défaut via `Intl.DateTimeFormat().resolvedOptions().locale` (aucune dépendance native nécessaire), avec surcharge explicite utilisateur persistée.
- **Plancher d'accessibilité** :
  - `accessibilityLabel`/`accessibilityRole` sur chaque élément interactif
  - Contraste WCAG AA (4.5:1 texte normal / 3:1 grand texte gras) vérifié sur **chaque paire de couleurs réellement utilisée**, pas juste les couleurs de marque isolées — un accent saturé peut ne passer qu'en grande taille/gras et exiger un texte plus foncé par-dessus, jamais blanc
  - Taille de police dynamique respectée via `maxFontSizeMultiplier` plutôt qu'ignorée
  - Cibles tactiles ≥44×44pt
- **Icônes** : préférer le rendu natif de la plateforme (expo-symbols/SF Symbols iOS, mappé vers Material Symbols en fallback Android/web) au lieu d'un set maison en PNG pour le chrome UI — hérite gratuitement du bon poids/échelle/comportement système et paraît authentiquement natif. Réserver l'illustration/emoji personnalisés à la marque (mascotte/logo) et à tout symbole intrinsèque au produit qui doit rester du texte Unicode (ex. une sortie qui doit être collable ailleurs comme texte) — ne jamais rehabiller ceux-là en images.
- **Design mobile actuel** : conventions type "Liquid Glass" iOS (matériaux réservés à la couche navigation/onglets, pas enfouis dans le contenu), micro-interactions/haptique liées à de vraies actions (pas de décoration gratuite), police système par défaut pour le corps/contrôles avec au plus une police de marque custom réservée au logotype (garde le bundle léger, évite les conflits avec la mise à l'échelle d'accessibilité).

## Composants natifs avec @expo/ui

`@expo/ui` expose de vrais composants **SwiftUI** (`@expo/ui/swift-ui`, iOS) et **Jetpack Compose** (`@expo/ui/jetpack-compose`, Android) — pas une imitation stylée en JS/Reanimated. Il existe aussi une version cross-plateforme plus limitée (import `@expo/ui` tout court, "Universal") mais dès que l'utilisateur demande "de vrais boutons/contrôles Apple", utiliser la version SwiftUI directe : c'est elle seule qui expose `buttonStyle('glass' | 'glassProminent')` — le vrai Liquid Glass iOS 26 (dégradé propre sur les versions antérieures géré par la lib, pas à gérer soi-même).

**Convention obligatoire — fichiers `.ios.tsx`/`.android.tsx`, jamais de `Platform.OS` runtime pour ça** : créer `MonComposant.ios.tsx` (import SwiftUI) à côté de `MonComposant.tsx` (repli JS cross-plateforme, utilisé par défaut sur Android/web, même nom de fichier sans suffixe). Metro résout automatiquement le bon fichier à la compilation selon la plateforme cible. Un `if (Platform.OS === 'ios')` runtime dans un fichier unique laisserait quand même l'import SwiftUI se faire bundler côté Android/web et risquerait un échec au chargement du module, pas juste au rendu — l'extension de fichier est la vraie garde, pas un test conditionnel.

**API de base (SwiftUI)** :
```tsx
import { Button, Host } from '@expo/ui/swift-ui';
import { accessibilityLabel, buttonStyle, controlSize, disabled, frame, tint } from '@expo/ui/swift-ui/modifiers';

<Host style={{ flexGrow: 1, flexBasis: 0 }} matchContents={{ horizontal: false, vertical: true }}>
  <Button
    label="Valider"
    systemImage="checkmark"          // nom SF Symbol brut, pas un composant Icon custom
    onPress={() => {/* ... */}}
    modifiers={[
      buttonStyle('glassProminent'), // 'bordered' | 'borderedProminent' | 'plain' | 'glass' | 'glassProminent'
      controlSize('large'),          // 'mini' | 'small' | 'regular' | 'large' | 'extraLarge'
      tint('#6D5AE6'),
      disabled(false),
      frame({ minHeight: 44 }),      // cible tactile HIG — aucun minimum garanti par défaut
      accessibilityLabel('Valider le formulaire'),
    ]}
  />
</Host>
```
- **`Host` est obligatoire** autour de tout élément `@expo/ui/swift-ui` — c'est le pont RN↔SwiftUI, rien ne s'affiche sans lui. `matchContents={{ horizontal: false, vertical: true }}` = hauteur qui suit le contenu du bouton, largeur qui remplit le parent flex — c'est ce qui permet une rangée de boutons à largeur égale en posant `flexGrow`/`flexBasis` sur le `Host`, pas sur le `Button` lui-même.
- Le style passe **exclusivement par le tableau `modifiers`** (fonctions composables), pas par des props directes : `buttonStyle`, `controlSize`, `tint`, `disabled`, `frame`, `padding`, `accessibilityLabel`/`accessibilityHint`/`accessibilityHidden`/`accessibilityValue`.
- `systemImage` attend un nom SF Symbol strictement typé (union `SFSymbols7_0` du package `sf-symbols-typescript`, dépendance transitive — l'importer directement en type-only est sûr même hors package.json). Si le nom vient d'une table de correspondance dynamique (ex. mapping maison `IconName → nom SF Symbol`), TypeScript ne peut pas prouver l'appartenance à l'union depuis un lookup runtime : caster explicitement (`as SFSymbols7_0`) avec un commentaire expliquant que la table source est déjà vérifiée par ailleurs (`satisfies`).

⚠️ **Testable en Expo Go, jamais dans un aperçu web** : `@expo/ui` est "Included in Expo Go" — aucun build EAS nécessaire pour tester, juste `expo start` + l'app Expo Go sur un vrai iPhone. Mais un rendu SwiftUI ne s'affiche PAS dans un aperçu web/navigateur (le fichier `.ios.tsx` n'est même pas inclus dans le bundle web). Ne jamais prétendre avoir vérifié visuellement un composant `@expo/ui/swift-ui` via un aperçu web ou un simulateur indisponible — le dire explicitement et demander à l'utilisateur de confirmer sur son propre appareil.

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
- [resources/revenuecat-api-v2.md](resources/revenuecat-api-v2.md) — cheatsheet API v2 RevenueCat (curl) : entitlements, offerings, produits, packages, clé Achat intégré
