# Contribuer

Ce skill est alimenté par des **leçons vécues** sur de vrais projets Expo/React Native, pas par de la documentation générique recopiée. C'est ce qui le rend utile — merci de garder cet esprit dans toute contribution.

## Ce qui fait une bonne contribution

- Un piège ou une procédure **réellement rencontrée** sur un projet, pas une supposition ou une extrapolation depuis la doc officielle.
- Une commande **réellement exécutée** avant d'être documentée — jamais une commande "qui devrait marcher".
- Un numéro de version, un prix ou une règle Apple/Google **vérifié à la date de la contribution**, avec la date ou un lien source explicite si l'info est susceptible de changer.
- Une correction d'une affirmation devenue fausse (SDK, tarifs, règles de store) — avec la source qui a permis de la détecter.

## Ce qui n'a pas sa place ici

- Des détails propres à un projet précis (bundle ID, nom d'app, couleurs de marque) — ce skill doit rester générique. Si un exemple concret aide à comprendre, le labelliser clairement comme "exemple vécu", jamais comme la norme.
- Des instructions supposant un Mac/Xcode disponible.
- Des numéros de version ou des prix inventés/devinés "à peu près".

## Processus

1. Ouvrir une issue ou une PR décrivant le contexte (quel projet, quelle situation a révélé le piège).
2. Si la PR modifie un comportement documenté, mettre à jour la documentation concernée **dans le même commit** (jamais "à faire plus tard").
3. Ajouter une entrée dans [CHANGELOG.md](CHANGELOG.md).
4. Les liens externes cités doivent être vérifiés avant la PR (pas de lien mort, pas de doc obsolète).

## Style

- Français, ton direct, phrases actionnables plutôt que descriptives.
- Expliquer le jargon technique à son premier usage.
- Toujours préciser explicitement quand une information est susceptible de changer (versions, prix, règles de plateforme) plutôt que de l'écrire comme une vérité figée.
