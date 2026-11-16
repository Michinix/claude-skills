# plane-gitlab-flow

## Vue d'ensemble

Skill Claude Code qui automatise le flow de dev Plane + GitLab : de l'issue Plane jusqu'à la MR GitLab.

**Pourquoi ce skill.** Chaque tâche demandait la même série de gestes manuels dans deux outils : renseigner l'issue Plane (assignation, dates, estimation, labels), créer la branche GitLab, commiter, pousser, rédiger la MR (description, labels, assignee, relecteur), puis revenir dans Plane. Ces allers-retours n'apportent aucune valeur au code et se répètent à chaque tâche : le skill les prend en charge, et le temps gagné sert à développer et à se concentrer sur des tâches réellement utiles.

## Flow : 2 phases

| Phase | Action |
|---|---|
| `start` | Crée l'issue Plane si elle n'existe pas (titre = user story de 15-20 mots, description, priorité, labels, estimation). Assigne l'issue à toi (demande avant d'écraser quelqu'un d'autre), dates début/fin = aujourd'hui, estimation Fibonacci auto, labels (+ `gitlab` en dernier). Attend la sync GitLab (2 min max), pose l'état "En cours" (la sync le remet en "Sprint Backlog"), crée la branche `<iid>-<titre>` |
| (dev) | Tu codes |
| `mr` | **Propose un message de commit et attend ta validation**, commite, pousse, puis crée la MR (ou la met à jour si elle existe) : titre = titre du commit, description en français + `Closes #<iid>` en dernière ligne, assignee, reviewers, labels de l'issue + `plane`. Passe l'issue Plane en "Test" (puis vérifie que la sync GitLab ne l'a pas changé). Prépare les captures dans `claude-screens/` |

Sans argument, le skill déduit la phase (branche courante, MR existante). `/plane-gitlab-flow status` indique la suite.

**À faire à la main dans Plane après la MR** : saisir le temps passé et ajouter les captures de `claude-screens/`. Le skill ne touche à Plane que pour l'état "Test".

**États Plane** : lus via `list_states` au début de chaque phase (nom, sinon groupe Plane ; demande si ambigu). Un projet qui renomme ses colonnes fonctionne sans config.

### Branches

- Nom : celui que GitLab génère pour l'issue, `<iid>-<titre>` tronqué à 100 caractères au dernier mot entier (ex. `42-en-tant-qu-utilisateur-je-veux-exporter-les-commandes-en-csv`). Le numéro est l'`iid` GitLab réel, lu dans `external_id` de l'issue Plane (`gl_12_42`), jamais calculé ni cherché par titre. La branche est créée côté GitLab, puis récupérée en local.
- Base : la branche de ta MR ouverte la plus récente ; les MR des autres sont ignorées. Aucune MR ouverte : `staging`. **Toute MR cible la branche par défaut du repo GitLab**, même empilée. Attention : une MR empilée affiche aussi les commits de sa MR parente jusqu'au merge de celle-ci.

### Labels et estimation

- Labels Plane choisis automatiquement parmi ceux de la config (champ `when`), traduits vers le label GitLab exact (casse et orthographe). Le label `plane` est obligatoire sur toutes les MR. Le skill ne crée jamais de label GitLab.
- Estimation : 1, 2, 3, 5, 8, 13, 21, déduite de l'issue, du contexte projet et des issues voisines. Annoncée avec sa raison, modifiable ensuite dans Plane.

## Prérequis

- [Claude Code](https://claude.com/claude-code)
- **MCP Plane** et **MCP GitLab** connectés et authentifiés (`/mcp`), en connecteurs claude.ai ou en serveurs MCP ajoutés à la main (le skill détecte le préfixe ; `allowed-tools` ne couvre que les connecteurs claude.ai, sinon ajouter les outils dans `.claude/settings.json`). Aucune clé d'API.
- Intégration GitLab active sur le projet Plane. Repo avec un remote GitLab.
- Optionnel : MCP Playwright, pour prendre les captures.

Le skill vérifie ces prérequis au début de chaque phase et s'arrête avec l'étape de connexion si l'un manque.

## Installation

Le skill est intégré au code de chaque projet, dès le début : copier le dossier dans `<projet>/.claude/skills/plane-gitlab-flow/` et le commiter. Rien d'autre à configurer.

## Structure

```
plane-gitlab-flow/
├── SKILL.md           point d'entrée : choix de la phase, prérequis, règles
├── phases/            start.md, mr.md
├── references/        config.md (clés, setup, labels), mcp-tools.md, troubleshooting.md
└── config.yml         générée au premier lancement, versionnée avec le projet
```

## Utilisation

```
/plane-gitlab-flow start PROJ-45
# ... dev ...
/plane-gitlab-flow mr
```

Ou en langage naturel : « crée l'issue pour… », « je prends la tâche », « j'ai fini », « fais la MR ».

## Configuration

`config.yml`, **générée automatiquement** au premier lancement via les MCP Plane et GitLab. Aucun fichier à copier ni à remplir. Détail et exemple : [`references/config.md`](references/config.md).

| Clé | Rôle (défaut) |
|---|---|
| `gitlab.project` | Projet GitLab, détecté depuis le remote git |
| `gitlab.stagingBranch` | Base quand tu n'as aucune MR ouverte (`staging`, sinon branche par défaut) |
| `gitlab.mrExtraLabels` | Labels posés sur toutes les MR, obligatoires (`[plane]`) |
| `labels` | Pour chaque label Plane : label GitLab exact (`gitlab`) et quand l'utiliser (`when`). Gère doublons de casse et orthographes différentes |
| `defaults.reviewers` | Relecteurs des MR (demandés une fois si absents) |
| `defaults.language` | Langue des textes publiés, MR (`français`) |

Fixe, non configurable : label de sync `gitlab`, rôles d'états Plane (Backlog / Sprint Backlog / En cours / Test, noms réels lus dans Plane), squash et suppression de branche activés, assignee = toi.

## Dépannage

Erreurs courantes et actions : [`references/troubleshooting.md`](references/troubleshooting.md).

## Limites

Règles complètes dans [`SKILL.md`](SKILL.md) (section *Règles*). En bref : commit toujours validé par toi, jamais de push force ni de merge, issue jamais en "Terminé"/"Annulé". Après la MR, temps passé et captures sont à faire à la main (le MCP Plane ne sait pas envoyer un fichier local).
