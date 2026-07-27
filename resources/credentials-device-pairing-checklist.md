# Checklist — login EAS, credentials Apple, enregistrement d'appareil

À suivre dans l'ordre. Chaque étape a un piège connu — ne la saute pas même si elle semble triviale.

## 1. Login EAS non-interactif

Un compte Expo créé via "Continuer avec Google" **n'a pas de mot de passe**, donc `eas login` classique échoue.

1. Demander à l'utilisateur d'aller sur https://expo.dev/settings/access-tokens et de générer un jeton d'accès personnel.
2. Stocker ce jeton **hors du dépôt git**, dans un fichier du profil utilisateur (jamais dans `.env` versionné, jamais commité).
3. L'utiliser pour toutes les commandes EAS non-interactives :
   ```powershell
   $env:EXPO_TOKEN = "le-token-ici"
   ```
4. Vérifier que ça marche : `eas whoami`

## 2. Initialisation du projet EAS

```powershell
eas init --force --non-interactive
```

- Si un ancien projet EAS existe déjà (reste d'une tentative précédente, ex. un essai no-code abandonné), **ne pas le réutiliser silencieusement** — le signaler explicitement à l'utilisateur et demander s'il veut repartir sur un projet propre.

## 3. Compte Apple Developer — le retrouver

L'Apple ID (compte développeur payant, ~99$/an) **n'est pas** le compte Expo/Google utilisé pour EAS. Pour le retrouver, dans l'ordre :

1. Réglages iPhone → nom en haut de l'écran → email affiché.
2. Chercher l'email de reçu "Apple Developer Program" dans la boîte mail.
3. Tester la connexion sur https://appstoreconnect.apple.com — si une app TestFlight existante apparaît, c'est le bon compte.

⚠️ Si "Invalid username and password combination" se répète : **arrêter d'essayer** (risque de blocage temporaire du compte). Retrouver d'abord le bon identifiant via les étapes ci-dessus. Si le mot de passe est perdu, utiliser https://iforgot.apple.com plutôt que de réessayer en boucle.

## 4. Enregistrement d'un appareil iOS (`eas device:create`)

Cette commande est **entièrement interactive** — impossible de scripter avec des réponses en pipe (erreur "Input is required, but stdin is not readable").

Deux façons de procéder :
- **Guider l'utilisateur** question par question dans son propre terminal (le plus simple si aucun outil interactif n'est disponible).
- **Piloter les prompts directement** si un outil de terminal interactif (type process interactif) est disponible.

### ⚠️ Piège du délai anti-vol iOS (le plus coûteux en temps si raté)

La méthode par défaut (scanner un QR code / visiter un lien web pour installer un profil de provisioning) peut déclencher un **"Security Delay" d'1 heure** — mais ce n'est pas un comportement universel de tous les iPhone : c'est spécifiquement le fait de la **Protection contre le vol d'appareil** (Stolen Device Protection, iOS 17.3+), une option qu'Apple recommande fortement mais qui n'est pas activée par défaut sur tous les téléphones. Le délai ne se déclenche que si cette option est activée **ET** que l'iPhone est hors d'un lieu "familier" (domicile/travail habituel) au moment de l'installation du profil. Demande à l'utilisateur s'il a activé cette protection (Réglages → Face ID et code → Protection contre le vol d'appareil) avant de t'attendre à ce piège précis — s'il ne l'a pas activée, l'installation via QR/web passe normalement sans délai.

**Si le délai se déclenche : rescanner le QR code remet le compteur à zéro.** Ne jamais faire rescanner l'utilisateur en pensant "relancer" la procédure — ça repart de 60 minutes.

**Contournement fiable sous Windows, qu'il y ait délai ou non (et donc à privilégier par défaut) :**
1. Brancher l'iPhone en USB.
2. Ouvrir l'app "Appareils Apple" (disponible sur Microsoft Store) ou iTunes.
3. Cliquer plusieurs fois sur la ligne d'informations sous le nom de l'appareil (numéro de série par défaut) jusqu'à voir apparaître "UDID".
4. Copier l'UDID.
5. Dans `eas device:create`, choisir l'option **"Input — allows you to type in UDIDs"** (pas la méthode QR/web) et coller l'UDID.

Cette méthode passe par le simple couple confirmation Face ID/code + "faire confiance à cet ordinateur" lors du branchement — une action ponctuelle, pas la "Security Delay" d'1h qui ne vise que des changements de réglages de sécurité sensibles (mot de passe Apple, désactivation de Find My, etc.). Elle évite donc le risque de délai dans tous les cas, activé ou non.

## 5. Contrats et déclarations bloquants (souvent la cause d'un échec de build silencieux)

Erreur typique : `Failed to register bundle identifier... agreement updates that must be resolved`.

À vérifier au navigateur (jamais résolvable en CLI) :
1. **Contrat "Apple Developer Program License Agreement"** à accepter sur https://developer.apple.com/account (souvent après un renouvellement annuel ou une mise à jour de contrat Apple).
2. **Déclaration de statut "trader"** (obligation Digital Services Act de l'UE, articles 30-31) à remplir sur https://appstoreconnect.apple.com. Ce n'est pas une formalité optionnelle : depuis le 17 février 2025, toute app distribuée commercialement dans l'UE (payante, avec IAP, ou toute autre activité commerciale) **doit** avoir ce statut déclaré et vérifié — sans ça, Apple **retire automatiquement l'app de la vente dans les 27 pays de l'UE**, y compris une app déjà publiée et qui tournait bien jusque-là. Seul un développeur individuel non-commercial distribuant une app strictement gratuite peut légitimement se déclarer non-trader.

⚠️ Le statut "trader" rend les coordonnées (adresse, email) **publiques** sur la fiche App Store européenne de l'app. Proposer à l'utilisateur d'utiliser une adresse/email dédiés professionnels plutôt que personnels s'il en a la possibilité.

## 6. Certificat de distribution

Un certificat de distribution appartient à **l'équipe Apple**, pas à un projet EAS précis. Réutiliser un certificat existant (même créé lors d'une ancienne tentative sur un autre projet) est normal et attendu.

Quand EAS demande "Reuse this distribution certificate?" → répondre **oui**.

## 7. Bundle ID — décision utilisateur, jamais un défaut

Si l'app existe déjà en TestFlight ou sur l'App Store :
- **Réutiliser exactement le même bundle ID** → conserve la fiche, les avis, les testeurs TestFlight, l'historique.
- **En créer un nouveau** → repart de zéro (nouvelle fiche, zéro avis, nouveaux testeurs à réinviter).

C'est une vraie décision produit à poser explicitement à l'utilisateur, jamais un choix à faire seul.

## 8. Configuration app.json requise

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "ITSAppUsesNonExemptEncryption": false
      }
    }
  }
}
```
Mettre `false` si l'app n'utilise pas de chiffrement propriétaire non-exempté (cas standard pour la plupart des apps), sinon `true` + documentation de conformité. Sans ce champ, le build échoue ou App Store Connect affiche un avertissement de config manquante.
