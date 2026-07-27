# Dépannage

Ce document indexe les pièges déjà documentés en détail ailleurs dans le skill, pour qu'on les retrouve vite. Le détail de chaque procédure reste dans son fichier source (une seule source de vérité par sujet) — ce document ne fait que pointer dessus.

| Symptôme | Cause probable | Où trouver la procédure |
|---|---|---|
| `eas login` échoue en boucle | Compte Expo créé via Google, pas de mot de passe | `SKILL.md` §EAS, [resources/credentials-device-pairing-checklist.md](../resources/credentials-device-pairing-checklist.md) §1 |
| `eas device:create` erreur "Input is required, but stdin is not readable" | Commande entièrement interactive, non scriptable en pipe | [resources/credentials-device-pairing-checklist.md](../resources/credentials-device-pairing-checklist.md) §4 |
| Installation d'un profil bloquée avec un délai d'1h | Protection contre le vol d'appareil activée + iPhone hors lieu familier | [resources/credentials-device-pairing-checklist.md](../resources/credentials-device-pairing-checklist.md) §4 |
| "Invalid username and password combination" répété sur Apple ID | Mauvais compte utilisé (Apple ID ≠ compte Expo/Google) | [resources/credentials-device-pairing-checklist.md](../resources/credentials-device-pairing-checklist.md) §3 |
| Build échoue avec "agreement updates that must be resolved" | Contrat Apple Developer Program non accepté et/ou statut trader DSA manquant | [resources/credentials-device-pairing-checklist.md](../resources/credentials-device-pairing-checklist.md) §5 |
| App déjà publiée retirée de la vente dans l'UE | Statut trader DSA non déclaré/vérifié (obligatoire depuis le 17/02/2025) | [resources/credentials-device-pairing-checklist.md](../resources/credentials-device-pairing-checklist.md) §5 |
| Build EAS échoue sans raison évidente liée au chiffrement | `ios.infoPlist.ITSAppUsesNonExemptEncryption` manquant dans `app.json` | [resources/credentials-device-pairing-checklist.md](../resources/credentials-device-pairing-checklist.md) §8 |
| RevenueCat ne valide pas les reçus d'achat | Mauvaise clé App Store Connect utilisée (clé générale au lieu de la clé "Achat intégré" dédiée) | `SKILL.md` §RevenueCat, [resources/revenuecat-api-v2.md](../resources/revenuecat-api-v2.md) |
| Clé "Achat intégré" perdue, impossible à récupérer | Le fichier `.p8` ne se télécharge qu'une seule fois | `SKILL.md` §RevenueCat |
| Erreur `parameter_error` sur un produit RevenueCat réel | Champ `subscription.duration` envoyé pour un produit App Store réel (réservé aux produits Test Store) | [resources/revenuecat-api-v2.md](../resources/revenuecat-api-v2.md) |
| Expo Go refuse d'ouvrir le projet ("unsupported SDK version") | Expo Go publié en retard sur le SDK du projet | `SKILL.md` §Aperçu live sur téléphone |
| Rejet App Store "site web repackagé" | Guideline 4.2 Minimum Functionality | [resources/appstore-submission-checklist.md](../resources/appstore-submission-checklist.md) |

Si un piège rencontré en pratique n'apparaît pas ici, c'est une bonne candidate de contribution — voir [../CONTRIBUTING.md](../CONTRIBUTING.md).
