# Installation

## Installation personnelle (tous les projets futurs)

Les skills Claude Code installés dans le dossier utilisateur `~/.claude/skills/` sont disponibles dans **toutes** les sessions Claude Code, quel que soit le dossier de travail.

```powershell
git clone https://github.com/RAAAAAGEEEEE/expo-eas-solo-dev.git "$env:USERPROFILE\.claude\skills\expo-eas-solo-dev"
```

Vérifier que Claude Code le détecte : démarrer une nouvelle session et demander la liste des skills disponibles, ou simplement lancer une conversation mentionnant "EAS build" — le skill doit apparaître comme disponible.

## Installation par projet uniquement

Pour ne l'activer que sur un projet précis plutôt que globalement, cloner dans `.claude/skills/` **à la racine du projet** au lieu du dossier utilisateur :

```powershell
git clone https://github.com/RAAAAAGEEEEE/expo-eas-solo-dev.git ".claude\skills\expo-eas-solo-dev"
```

## Mettre à jour une installation existante

```powershell
cd "$env:USERPROFILE\.claude\skills\expo-eas-solo-dev"
git pull origin main
```

## Désinstaller

Supprimer simplement le dossier — aucune trace ailleurs sur le système (le skill ne modifie rien en dehors de son propre dossier).

```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\expo-eas-solo-dev"
```
