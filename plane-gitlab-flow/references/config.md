# Config par projet

Fichier : `config.yml`, à côté de `SKILL.md`. Clé absente → défaut. Valeurs fixes du skill, non configurables : label de sync `gitlab`, rôles d'états Plane « Backlog », « Sprint Backlog », « En cours », « Test » (résolus à chaque phase via `list_states`, voir `SKILL.md`), `squash` et suppression de la branche source activés, assignee = utilisateur courant.

| Clé | Rôle (défaut) |
|---|---|
| `gitlab.project` | chemin du projet GitLab, détecté depuis le remote git (`acme/app`) |
| `gitlab.stagingBranch` | base quand je n'ai aucune MR ouverte (`staging`, sinon branche par défaut du repo) |
| `gitlab.mrExtraLabels` | labels posés sur **toutes** les MR, obligatoires (`[plane]`) |
| `labels` | label Plane → `{ gitlab, when }` : label GitLab exact et quand l'utiliser |
| `defaults.reviewers` | relecteurs des MR (demandés une fois si absents) |
| `defaults.language` | langue des textes publiés, MR (`français`) |

## Setup automatique (fichier absent)

Ne rien demander de préalable, ne jamais renvoyer l'utilisateur copier un fichier.

1. `gitlab.project` : remote git (`git remote get-url origin`) + `get_project`. Branche `staging` absente de `origin` → `stagingBranch` = branche par défaut.
2. `labels` : lister les labels Plane (projet de l'issue) et GitLab (`search_labels`), apparier par nom insensible à la casse et aux accents, écrire un `when` par label.
3. Label Plane sans correspondance dans GitLab : **ne jamais le créer**. Demander (orthographe différente, doublon de casse) ou, si rien ne correspond, laisser `gitlab: null` et le dire.
4. `defaults.reviewers` : demander une fois (proposer les membres humains du projet, sans bots).
5. Écrire `config.yml`, afficher un résumé, enchaîner sur la phase demandée. Fichier versionné avec le projet : ne pas l'exclure de git, ne pas commiter à la place de l'utilisateur.

## Labels

Les labels GitLab existants font foi. **Le skill ne crée jamais de label GitLab**, ni au setup ni ailleurs.

- **Plane (choix au `start`)** : label métier choisi parmi les clés de `labels`. Le label `Plane` n'est jamais posé par le skill.
- **GitLab (MR)** : label Plane présent dans `labels` → son `gitlab` tel quel (`null` = aucun). `gitlab.mrExtraLabels` → forme exacte de la config (`plane`, jamais `Plane`).
- **Label absent de `labels`** : chercher dans les labels GitLab (casse et accents ignorés), prendre l'orthographe exacte. Plusieurs correspondances (`Bug`/`bug`) → celle de l'issue GitLab, sinon demander. Aucune → ignorer et le dire.
- La sync Plane → GitLab recopie les labels **par nom** : un nom différent n'est pas recopié (ex. `Dépendances` / `Dependencies`). Le tableau `labels` sert à poser le bon label sur la MR.

## Exemple

```yaml
gitlab:
  project: acme/app
  stagingBranch: staging
  mrExtraLabels: [plane]
labels:
  Bug:         { gitlab: Bug,          when: correction d'un comportement cassé }
  Dépendances: { gitlab: Dependencies, when: mise à jour ou ajout de packages }
defaults:
  reviewers: [relecteur1]
  language: français
```
