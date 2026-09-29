# Limites

Honnêtes et non minimisées, comme l'exige le standard de documentation de ce dépôt.

## Couverture d'expérience

- Ce skill est construit à partir d'**un seul projet réel de bout en bout** : Mysterymoji, une app **iOS**. Le contenu iOS/EAS/Apple est donc beaucoup plus éprouvé que le contenu Android/Play Store, qui repose sur de la documentation vérifiée (règle des 12 testeurs, Play App Signing) mais n'a pas encore été traversé sur un cycle de publication complet.
- Certaines procédures sont documentées à partir de sources officielles sans avoir été exécutées par l'auteur — notamment la recomposition de captures d'écran App Store et la publication Play Console. Le texte le signale là où c'est le cas.

## Évaluations non automatisées

`evals/evals.json` contient cinq cas (prompt, sortie attendue, expectations vérifiables) au format `evals.json` de skill-creator. Ils n'ont pas été exécutés par un harnais : aucun résultat chiffré n'est publié. La vérification repose donc surtout sur :
- la relecture humaine des cas contre la réponse du skill ;
- des audits ponctuels avec recherche web pour vérifier les faits datés (voir CHANGELOG) ;
- l'usage réel sur de nouveaux projets, qui remonte les erreurs.

## Péremption des faits datés

Versions de SDK, prix (Apple Developer Program, Google Play, plans EAS), quotas de build, et règles de conformité Apple/Google **changent régulièrement**. Le skill instruit explicitement de toujours revérifier plutôt que de faire confiance à une valeur mémorisée — mais le contenu écrit dans ce dépôt, lui, reflète l'état constaté à la date de son dernier audit (voir `CHANGELOG.md`). Une information vue ici peut être devenue fausse depuis.

## Environnement supposé

Les commandes sont écrites pour **PowerShell sous Windows**. Utilisable comme référence sur macOS/Linux, mais les commandes devront être adaptées (syntaxe shell, chemins).

## Pas de garantie de conformité store

Ce skill aide à anticiper les motifs de rejet les plus courants (Guideline 4.2, IAP, DSA) mais ne garantit aucune approbation App Store/Play Store — les règles de revue évoluent et chaque cas est jugé individuellement par les équipes de revue Apple/Google.
