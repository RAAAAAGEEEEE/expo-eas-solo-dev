# Architecture

## Principe : divulgation progressive

Ce skill suit le modèle standard des skills Claude Code à trois niveaux de chargement :

1. **Métadonnées** (`name` + `description` du frontmatter YAML de `SKILL.md`) — toujours en contexte, sert au déclenchement automatique.
2. **Corps de `SKILL.md`** — chargé en contexte dès que le skill se déclenche. Contient la posture générale et les points essentiels de chaque sujet, avec des renvois vers les ressources détaillées.
3. **`resources/*.md`** — chargés à la demande seulement, quand Claude Code juge nécessaire d'entrer dans le détail d'une procédure précise.

Ce découpage évite de saturer le contexte de Claude Code avec des checklists complètes (ex. la procédure pas-à-pas de pairing d'un device iOS) alors que 80% des conversations n'en ont besoin que d'un résumé.

## Fichiers

```
expo-eas-solo-dev/
├── SKILL.md                                        # posture + résumé actionnable de chaque sujet
├── resources/
│   ├── eas.json.template                           # template eas.json commenté
│   ├── credentials-device-pairing-checklist.md      # procédure pas-à-pas EAS/Apple/device
│   ├── appstore-submission-checklist.md             # checklist de soumission App Store/Play Store
│   └── revenuecat-api-v2.md                         # cheatsheet API v2 RevenueCat (curl)
├── README.md, LICENSE, CHANGELOG.md, CONTRIBUTING.md, SECURITY.md
└── docs/                                            # ce dossier — documentation du skill lui-même
```

## Pourquoi le contenu est organisé par sujet, pas par phase de projet

Une app passe par les mêmes sujets (stack, EAS, conformité store, i18n, tests) à des moments différents selon le projet — certains commencent par la conformité (produit sensible), d'autres par la stack. Organiser `SKILL.md` par sujet plutôt que par "Phase 1, Phase 2..." permet à Claude Code de piocher directement la section pertinente au moment où la conversation l'exige, sans dépendre d'un ordre linéaire supposé.

## Où va une nouvelle leçon

- **Piège ponctuel avec une procédure longue** (plus de 5-6 étapes, ou qui casse rarement mais coûte cher en temps si raté) → nouveau fichier ou section dans `resources/`.
- **Point qui doit influencer une décision à chaque conversation** (règle à ne jamais oublier, comportement par défaut à adopter) → corps de `SKILL.md`.
- **Changement de comportement du skill lui-même** (nouvelle section, contenu retiré) → entrée dans `CHANGELOG.md` dans le même commit.
