# claude-skills

Collection de skills Claude. Un dossier par skill, chacun avec un `SKILL.md`.

## Skills

| Skill | Rôle |
|---|---|
| [`plane-gitlab-flow`](plane-gitlab-flow/) | Flow Plane + GitLab autour du dev. `start` : issue Plane (création, assignation, dates, estimation, labels), sync GitLab, branche. `mr` : commit validé, push, MR, issue en « Test » |

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
