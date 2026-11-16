# claude-skills

Collection de skills [Claude Code](https://claude.com/claude-code). Un dossier par skill, chacun avec un `SKILL.md`.

## Skills

| Skill | Rôle |
|---|---|
| [`plane-gitlab-flow`](plane-gitlab-flow/) | Flow Plane + GitLab après le dev : issue, branche, MR, passage en Test |

## Installation

Un skill dans un projet :

```bash
mkdir -p .claude/skills
cp -r <skill> .claude/skills/
```

Un skill pour tous les projets :

```bash
cp -r <skill> ~/.claude/skills/
```

Seuls les fichiers du dossier du skill comptent (`SKILL.md` et ses fichiers annexes). Le `README.md` racine, `.git` et `.gitignore` sont inutiles côté projet.

## Ajouter un skill

1. Créer `<nom-du-skill>/SKILL.md` avec le frontmatter `name` et `description`.
2. Ajouter un `README.md` dans le dossier du skill (usage, prérequis, config).
3. Lister le skill dans le tableau ci-dessus.
