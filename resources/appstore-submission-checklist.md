# Checklist — soumission App Store Connect (et équivalents Play Console)

À traiter comme une revue produit **avant** de coder la majorité des fonctionnalités, pas en rattrapage juste avant la soumission.

## Revue produit (à faire tôt, pas en rattrapage)

- [ ] **Guideline 4.2 "Minimum Functionality"** anticipée : l'app a plusieurs contenus/thèmes, pas une seule fonction mono-usage qui ressemblerait à "un site web repackagé". Mitigations concrètes à considérer :
  - [ ] Mode jeu/défi ou variation d'usage
  - [ ] Favoris / historique
  - [ ] Partage natif — texte ET image (ex. via react-native-view-shot)
  - [ ] Réglages avec de vrais interrupteurs (haptique, animations, apparence, langue)
  - [ ] Onboarding
- [ ] **Classification d'âge** décidée consciemment avec l'utilisateur (13+ vs "Conçu pour les enfants" 4+) — impacte les règles de paywall (parental gate COPPA si 4+). ⚠️ Le système a changé en 2025 (bandes 13+/16+/18+ à la place de 12+/17+) et le questionnaire App Store Connect comporte de nouvelles questions obligatoires (fonctionnalités sociales, contrôles in-app, thèmes médicaux/bien-être, violence) — sans réponses à jour, les soumissions **et** les mises à jour sont bloquées. Répondre au questionnaire actuel, ne jamais supposer qu'une classification existante est encore valide.
- [ ] **Répartition gratuit/premium** : la fonction cœur reste illimitée et gratuite ; le payant porte sur des à-côtés (contenus supplémentaires, quota quotidien élargi, favoris illimités, export sans filigrane, personnalisation).

## Monétisation (si IAP)

- [ ] **App Store Small Business Program activé** dans App Store Connect (commission 15 % au lieu de 30 %). Non automatique : sans inscription explicite, Apple prélève 30 %. Voir `SKILL.md` §Décisions à prendre au jour 1.
- [ ] RevenueCat (react-native-purchases) intégré — jamais Stripe/processeur externe pour du contenu numérique déblocable in-app (violerait la règle Apple 3.1.1).
- [ ] Bouton "Restaurer les achats" présent et fonctionnel.
- [ ] Champ code promo visible dans l'UI (même avant d'être branché à un vrai back-end — les Offer Codes Apple se configurent une fois un abonnement au statut "Ready to Submit" dans App Store Connect).

## Fiche App Store Connect

- [ ] **Icône** 1024×1024 PNG, sans transparence, sans coins arrondis pré-appliqués (l'OS masque lui-même les coins).
- [ ] Titre
- [ ] Sous-titre ≤ 30 caractères
- [ ] Description
- [ ] Mots-clés ≤ 100 caractères, séparés par des virgules
- [ ] Texte promotionnel ≤ 170 caractères
- [ ] Plan de captures d'écran mappé aux vrais écrans de l'app (pas de mockups qui ne correspondent pas au produit réel)
- [ ] Note au reviewer honnête : préciser si l'app fonctionne hors-ligne, comment tester un éventuel paywall, fournir un vrai code promo de test une fois le mécanisme en place.
- [ ] `ios.infoPlist.ITSAppUsesNonExemptEncryption` renseigné dans app.json (voir credentials-device-pairing-checklist.md §8).

### Captures d'écran App Store sans Mac — beaucoup plus simple qu'avant

⚠️ Beaucoup de tutoriels encore en ligne décrivent l'ancienne exigence (une série de captures par taille d'écran : 6.9", 6.5", 5.5", iPad...). **Ce n'est plus le cas** : Apple ne demande plus qu'**une seule série iPhone**, en 6.9" (1320 × 2868 px ; le 6.7" en 1290 × 2796 px est accepté en repli), et met lui-même à l'échelle pour tous les iPhone plus petits. Une série iPad 13" (2064 × 2752 px) s'ajoute uniquement si l'app supporte l'iPad. Ces dimensions évoluent avec les nouveaux modèles — revérifier sur la doc App Store Connect plutôt que de recopier ces valeurs les yeux fermés.

Conséquence pour un utilisateur sans Mac : le problème se réduit à produire **une seule série d'images à une seule résolution**, ce qui ne justifie plus de louer un Mac cloud. Deux approches :
- Si l'utilisateur possède un iPhone dont la capture native fait déjà la bonne taille (modèles Pro Max récents), ses captures d'écran brutes conviennent directement.
- Sinon, recomposer ses captures réelles dans un canevas à la résolution exacte (script d'édition d'image), ou utiliser un générateur de captures marketing — l'immense majorité des fiches App Store utilisent de toute façon des captures habillées (texte d'accroche, cadre), pas des captures brutes.

Contraintes à respecter : 1 à 10 captures par langue, PNG ou JPEG **aplati** (pas de transparence, pas de calque alpha). Les 2-3 premières sont celles visibles dans les résultats de recherche — y placer l'argument principal.

⚠️ Les captures doivent montrer l'app réelle. Une fiche dont les captures ne correspondent pas à ce que fait vraiment l'app est un motif de rejet classique.

## Prérequis "durs" avant Submit (pas avant)

- [ ] URL de politique de confidentialité — pas besoin d'exister en amont, mais bloquante à la soumission réelle. Rédigée honnêtement (ex. "nous ne collectons aucune donnée directement ; Apple/RevenueCat traitent les données d'achat" plutôt qu'un "zéro donnée" mensonger si ce n'est pas littéralement vrai).
- [ ] Contrat "Apple Developer Program License Agreement" accepté sur developer.apple.com/account.
- [ ] Déclaration de statut "trader" (DSA UE) remplie sur App Store Connect si applicable — coordonnées deviennent publiques, prévenir l'utilisateur avant qu'il saisisse des infos personnelles.

## Équivalents Google Play (si l'app est aussi sur Android)

### Bloquant à anticiper dès le jour 1 — pas à la soumission

- [ ] **Test fermé de 12 testeurs pendant 14 jours consécutifs** effectué, si le compte développeur est **personnel** et a été créé après le 13/11/2023. Compte organisation (entité légale) = exempté. Voir la section "Décisions à prendre au jour 1" de `SKILL.md` — ce point ajoute **~3 semaines** au calendrier et ne peut pas être rattrapé en fin de projet.

### Fiche et conformité

- [ ] Fiche Play Console : icône, captures d'écran, description courte/longue, catégorie.
- [ ] Politique de confidentialité renseignée dans Play Console (même exigence qu'Apple).
- [ ] Questionnaire de classification de contenu complété.
- [ ] Section "Sécurité des données" (Data safety) remplie honnêtement.
- [ ] Play Billing configuré via RevenueCat si IAP (même logique qu'iOS : jamais de processeur externe pour du contenu numérique).

### Signature de l'app — le point à ne pas rater

- [ ] **Play App Signing activé** (c'est le défaut recommandé pour tout nouveau projet).

Pourquoi c'est important à expliquer à un utilisateur non technique : une app Android est signée par une clé cryptographique, et **une app ne peut être mise à jour que si la nouvelle version est signée avec la même clé**. Perdre cette clé signifie ne plus jamais pouvoir mettre à jour l'app — il faudrait en republier une nouvelle, en perdant les installations et les avis.

Play App Signing supprime ce risque en distinguant deux clés :
- la **clé de signature de l'app** (app signing key), conservée par Google, jamais entre les mains de l'utilisateur ;
- la **clé d'upload** (upload key), celle qu'utilise EAS pour signer ce qu'il envoie à Google.

Si la clé d'upload est perdue ou compromise, elle peut être **réinitialisée** depuis la Play Console sans conséquence pour les utilisateurs. Sans Play App Signing, la perte de la clé unique est définitive et irréversible.

Côté EAS : `eas build` gère et stocke le keystore (la clé d'upload) automatiquement pour le compte de l'utilisateur — il n'a rien à manipuler à la main. Lui rappeler quand même que ces credentials sont récupérables via `eas credentials`, et qu'ils sont liés à son compte Expo.

## Si l'app est rejetée

Un rejet n'est pas un échec de projet — c'est un aller-retour courant, y compris pour des apps qui finissent approuvées sans changement. Réagir méthodiquement plutôt que dans l'urgence :

1. **Lire précisément quelle guideline est citée** dans le message du Resolution Center — le motif exact conditionne la réponse.
2. **Répondre dans le Resolution Center** (App Store Connect) plutôt que de faire un appel formel : dans l'immense majorité des cas, corriger le point soulevé ou fournir l'explication/la démonstration manquante est **plus rapide** qu'une procédure d'appel.
3. **N'appeler devant l'App Review Board** que si le reviewer a manifestement mal compris le concept de l'app, mal appliqué une guideline, ou ignoré des éléments déjà fournis dans les notes de revue. Le nombre d'appels est limité — les garder pour les cas solides.
4. **Ton professionnel et factuel**, avec des preuves concrètes (captures, identifiants de test, explication du fonctionnement). Jamais de contestation émotionnelle.
5. Compter en général 24 à 72 h pour une réponse, davantage pour une escalade au Board.

Réflexe préventif : une note au reviewer honnête et complète dès la première soumission (app fonctionne hors-ligne, comment tester le paywall, code promo de test valide) évite une bonne partie des rejets pour "impossible à évaluer".
