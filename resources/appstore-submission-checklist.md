# Checklist — soumission App Store Connect (et équivalents Play Console)

À traiter comme une revue produit **avant** de coder la majorité des fonctionnalités, pas en rattrapage juste avant la soumission.

## Revue produit (à faire tôt, pas en rattrapage)

- [ ] **Guideline 4.2 "Minimum Functionality"** anticipée : l'app a plusieurs contenus/thèmes, pas une seule fonction mono-usage qui ressemblerait à "un site web repackagé". Mitigations concrètes à considérer :
  - [ ] Mode jeu/défi ou variation d'usage
  - [ ] Favoris / historique
  - [ ] Partage natif — texte ET image (ex. via react-native-view-shot)
  - [ ] Réglages avec de vrais interrupteurs (haptique, animations, apparence, langue)
  - [ ] Onboarding
- [ ] **Classification d'âge** décidée consciemment avec l'utilisateur (13+ vs "Conçu pour les enfants" 4+) — impacte les règles de paywall (parental gate COPPA si 4+).
- [ ] **Répartition gratuit/premium** : la fonction cœur reste illimitée et gratuite ; le payant porte sur des à-côtés (contenus supplémentaires, quota quotidien élargi, favoris illimités, export sans filigrane, personnalisation).

## Monétisation (si IAP)

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

### Captures d'écran App Store sans Mac — piste non encore validée en pratique

Apple exige des captures aux résolutions exactes de chaque taille d'écran de référence (ex. iPhone 6.9", 6.5", iPad 13"...), normalement produites via le simulateur Xcode — indisponible sans Mac. Piste envisagée mais pas encore testée de bout en bout sur un vrai projet : recomposer les captures à partir de vraies photos d'écran prises par l'utilisateur sur son propre téléphone (via le bouton physique, taille native de son appareil), en les replaçant/redimensionnant dans un canevas à la résolution exacte attendue par Apple (via un script ou un outil d'édition d'image), plutôt que de louer un service de Mac cloud à l'heure. Avantage : zéro coût récurrent, aucun accès distant à configurer. Point à vérifier si ce cas se représente : est-ce qu'Apple valide des captures recomposées de cette façon (pas de règle explicite connue l'interdisant, mais à confirmer en conditions réelles) — documenter ici le résultat une fois éprouvé.

## Prérequis "durs" avant Submit (pas avant)

- [ ] URL de politique de confidentialité — pas besoin d'exister en amont, mais bloquante à la soumission réelle. Rédigée honnêtement (ex. "nous ne collectons aucune donnée directement ; Apple/RevenueCat traitent les données d'achat" plutôt qu'un "zéro donnée" mensonger si ce n'est pas littéralement vrai).
- [ ] Contrat "Apple Developer Program License Agreement" accepté sur developer.apple.com/account.
- [ ] Déclaration de statut "trader" (DSA UE) remplie sur App Store Connect si applicable — coordonnées deviennent publiques, prévenir l'utilisateur avant qu'il saisisse des infos personnelles.

## Équivalents Google Play (si l'app est aussi sur Android)

- [ ] Fiche Play Console : icône, captures d'écran, description courte/longue, catégorie.
- [ ] Politique de confidentialité renseignée dans Play Console (même exigence qu'Apple).
- [ ] Questionnaire de classification de contenu complété.
- [ ] Section "Sécurité des données" (Data safety) remplie honnêtement.
- [ ] Play Billing configuré via RevenueCat si IAP (même logique qu'iOS : jamais de processeur externe pour du contenu numérique).
