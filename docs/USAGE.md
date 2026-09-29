# Utilisation

## Déclenchement

Le skill se déclenche **automatiquement** — rien à invoquer explicitement. Claude Code le charge dès que la conversation touche à Expo/React Native, EAS, TestFlight, App Store Connect, RevenueCat, `@expo/ui`, ou décrit un problème typique du domaine (voir la liste complète dans le frontmatter de `SKILL.md`).

## Entrées attendues

Le skill part du principe que l'utilisateur :
- ne code pas lui-même (attend des commandes prêtes à copier-coller, pas des extraits de code à intégrer soi-même) ;
- n'a pas de Mac (Windows ou équivalent sans Xcode) ;
- peut avoir ou non déjà un compte Expo/Apple Developer/Google Play — le skill guide la découverte de ces comptes plutôt que de les supposer déjà en main.

## Sorties attendues

- Des commandes PowerShell avec chemins absolus, prêtes à exécuter.
- Un découpage explicite en phases, avec un point **STOP/GO** avant toute action consommant une ressource rare (build EAS) ou touchant un compte réel (Apple, Google, paiement).
- Une vulgarisation du jargon technique au moment où il apparaît, pas un cours à part.

## Exemples d'invocation

**Exemple 1 — démarrage de projet**
> Prompt utilisateur : *"Je veux créer une app de quiz culturel en Expo, je n'ai pas de Mac, je pars de zéro."*
> Comportement attendu : le skill guide le choix de la dernière version SDK stable (vérifiée, pas devinée), propose la baseline de dépendances, et alerte sur `expo-av` si mentionné dans un tutoriel suivi par l'utilisateur.

**Exemple 2 — blocage EAS**
> Prompt utilisateur : *"Mon build échoue avec une erreur sur un agreement Apple à résoudre."*
> Comportement attendu : le skill identifie la cause probable (contrat Apple Developer Program non accepté et/ou statut trader DSA manquant), renvoie vers `resources/credentials-device-pairing-checklist.md` §5, et précise que ça se résout uniquement au navigateur.

**Exemple 3 — device pairing**
> Prompt utilisateur : *"Comment enregistrer mon iPhone pour tester l'app ?"*
> Comportement attendu : le skill recommande directement la méthode UDID par câble USB (pas la méthode QR par défaut), avec les étapes exactes.

## Exemple de sortie type

```
Phase 2 — Enregistrement de ton iPhone

1. Branche ton iPhone en USB à cet ordinateur.
2. Ouvre l'app "Appareils Apple" (Microsoft Store) ou iTunes.
3. Clique plusieurs fois sur la ligne sous le nom de l'appareil jusqu'à voir "UDID".
4. Copie cet UDID.

Une fois que tu l'as, dis-le-moi (ou colle-le ici) et je lance :
  eas device:create
en choisissant l'option "Input — allows you to type in UDIDs".

⚠️ STOP avant de continuer : cette étape enregistre ton appareil auprès
d'Apple pour ce projet. Confirme que tu veux continuer.
```

## Comportements de repli

- Si une version SDK, un prix ou une règle Apple/Google n'est pas vérifiable dans l'instant, le skill le signale explicitement plutôt que d'inventer une valeur.
- Si `eas device:create` ne peut pas être piloté par un outil interactif, le skill bascule sur un guidage question par question dans le terminal de l'utilisateur.
- Si Expo Go installé ne supporte pas le SDK du projet, le skill propose trois options de repli (mise à jour d'Expo Go, `eas go`, simulateur iOS) plutôt que de bloquer.

## Fixtures et évaluations

Cinq cas d'évaluation sont dans [`evals/evals.json`](../evals/evals.json) : chacun donne un prompt, la sortie attendue et une liste d'expectations à vérifier contre la réponse du skill. Ils ne sont pas exécutés automatiquement, voir [LIMITATIONS.md](LIMITATIONS.md).
