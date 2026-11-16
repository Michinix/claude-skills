# Outils MCP (vérifiés en lecture sur une instance réelle)

Préfixes par défaut : `mcp__claude_ai_Plane__` et `mcp__claude_ai_Gitlab__` (connecteurs claude.ai, noms visibles dans `/mcp`). Autre connexion → autre préfixe, voir *Outils MCP* dans `SKILL.md`. Outils différés : un seul ToolSearch `select:` par phase avec la liste ci-dessous.

| Phase | Plane | GitLab |
|---|---|---|
| `start` | `get_me`, `list_projects`, `search_work_items`, `retrieve_work_item_by_identifier`, `create_work_item`, `update_work_item`, `manage_work_item_assignee`, `manage_work_item_label`, `list_states`, `list_labels`, `list_work_item_types`, `get_project_estimate`, `list_project_estimate_points` | `get_work_item`, `list_merge_requests`, `list_branches`, `add_branch`, `get_project` |
| `mr` | `retrieve_work_item_by_identifier`, `list_states`, `update_work_item` | `get_work_item`, `list_merge_requests`, `get_merge_request`, `save_merge_request`, `get_project`, `get_user`, `list_project_members` |
| setup | `list_labels` | `get_project`, `search_labels`, `list_project_members` |

## Lien Plane ↔ GitLab

- Après la sync, l'issue Plane porte `external_source` (ex. `GITLAB_ENTERPRISE`) et `external_id = gl_<id projet GitLab>_<iid>` (ex. `gl_12_42` → projet 12, issue `#42`). **Source de vérité** pour retrouver l'issue GitLab : jamais de recherche par titre.
- Côté GitLab, une note commence par `<!-- PLANE ISSUE LINKED REFERENCE COMMENT -->` et contient `[PROJ-123]` + lien Plane. Sert à retrouver l'issue Plane depuis une branche.
- Trois numéros différents : iid GitLab `42` ≠ identifiant Plane `PROJ-123` ≠ `REF 12` (ligne libre de la description, sans lien avec les deux autres).
- Délais mesurés : `external_id` renseigné ~7 s après le label `gitlab` ; la note de référence côté GitLab apparaît ~30 à 70 s après la création de l'issue GitLab (le sync ajoute aussi le label `plane` à l'issue GitLab) (d'où la demande de l'identifiant Plane si la note manque dans `mr`).
- Les commentaires GitLab sont recopiés dans Plane : le skill ne poste aucun commentaire.
- À la création de l'issue GitLab, le bot **remet l'état Plane dans le groupe « unstarted » (ex. « Sprint Backlog »)** ; assignee, dates, estimation et labels Plane sont conservés. Assignee et dates ne sont pas recopiés côté GitLab ; les labels le sont (par nom).
- Supprimer l'issue Plane ne supprime ni ne ferme l'issue GitLab.
- Plane peut assigner une personne par défaut à la création d'une issue (réglage du projet).
- Créer une MR lance la pipeline CI du projet.
- Aucun outil MCP ne supprime une branche ni une issue GitLab (seulement fermer l'issue).

## Plane

| Besoin | Outil | Piège |
|---|---|---|
| Moi | `get_me` | `id` = UUID à passer comme assignee |
| Projet | `list_projects` | préfixe (`PROJ`) → UUID `id` |
| Issue | `retrieve_work_item_by_identifier` (`PROJ-123`, `expand=assignees,labels,state`) | `id` = UUID de l'issue |
| États, labels, types | `list_states`, `list_labels`, `list_work_item_types` | `list_states` à chaque phase ; par nom puis par groupe, jamais d'UUID en dur |
| Mise à jour | `update_work_item` | `state` = UUID ; dates `YYYY-MM-DD` ; `priority` en minuscules |
| Assignee | `manage_work_item_assignee` (`add_user_id`, `remove_user_id`) | un utilisateur à la fois |
| Label | `manage_work_item_label` (`add_label_id`) | un seul à la fois |
| Estimation | `get_project_estimate` → `list_project_estimate_points` → `update_work_item(estimate_point=<UUID du point>)` | **pas** `point` (reste `null`) |
| Doublons | `search_work_items` | texte libre, tous projets : filtrer par préfixe |

## GitLab

| Besoin | Outil | Piège |
|---|---|---|
| Moi | `get_user` (`me: true`) | `username` pour assignee |
| Projet | `get_project` (`project_id` = chemin) | `default_branch`, `id` numérique |
| Issue | `get_work_item` (`work_item_iid`, `include: ["notes"]`) | labels dans le widget `LABELS` |
| Branche | `add_branch` (`branch`, `ref`) | crée côté GitLab ; ensuite `git fetch` + `git switch` |
| Mes MR | `list_merge_requests` (`scope: created_by_me`, `state: opened`) | |
| MR d'une branche | `list_merge_requests` + filtre sur `sourceBranch` | |
| Commits d'une MR | `get_merge_request` (`include: ["commits"]`) | ne renvoie pas les relecteurs |
| Créer / modifier une MR | `save_merge_request` | sans `merge_request_iid` = création ; avec = mise à jour |
| Labels | `search_labels` (`is_project: true`) | |
| Membres | `list_project_members` | ignorer les comptes bot (`group_*_bot_*`, `project_*_bot_*`) |
