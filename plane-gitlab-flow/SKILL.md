---
name: plane-gitlab-flow
description: Flow de dev Plane + GitLab, pour un projet relié à Plane et GitLab (config.yml présent, branche `<iid>-…`, ou issue Plane citée). start = crée l'issue Plane si besoin, assignation, état "En cours", dates, estimation, labels, branche. mr = commit proposé et validé, push, MR GitLab complète, issue Plane en "Test". Utiliser pour "crée l'issue", "je prends la tâche", "start", "j'ai fini", "fais la MR", "mr", ou /plane-gitlab-flow. Ne pas utiliser pour un simple commit ou push hors de ce flow.
allowed-tools: mcp__claude_ai_Plane__get_me mcp__claude_ai_Plane__list_projects mcp__claude_ai_Plane__search_work_items mcp__claude_ai_Plane__retrieve_work_item_by_identifier mcp__claude_ai_Plane__create_work_item mcp__claude_ai_Plane__update_work_item mcp__claude_ai_Plane__manage_work_item_assignee mcp__claude_ai_Plane__manage_work_item_label mcp__claude_ai_Plane__list_states mcp__claude_ai_Plane__list_labels mcp__claude_ai_Plane__list_work_item_types mcp__claude_ai_Plane__get_project_estimate mcp__claude_ai_Plane__list_project_estimate_points mcp__claude_ai_Gitlab__get_work_item mcp__claude_ai_Gitlab__list_merge_requests mcp__claude_ai_Gitlab__get_merge_request mcp__claude_ai_Gitlab__save_merge_request mcp__claude_ai_Gitlab__list_branches mcp__claude_ai_Gitlab__add_branch mcp__claude_ai_Gitlab__get_project mcp__claude_ai_Gitlab__get_user mcp__claude_ai_Gitlab__list_project_members mcp__claude_ai_Gitlab__search_labels Bash(git status:*) Bash(git diff:*) Bash(git log:*) Bash(git fetch:*) Bash(git switch:*) Bash(git add:*) Bash(git push -u origin:*) Bash(git remote get-url:*) Bash(git rev-parse:*) Bash(date:*)
---

# Plane + GitLab flow

Deux phases : `start` (issue Plane prête, sync GitLab, branche) et `mr` (commit validé, push, MR, issue Plane en « Test »). Le temps passé et les captures se font **à la main** dans Plane.

## Choisir la phase

Argument donné (`start`, `mr`, `status`) → cette phase. Sinon déduire : branche courante, MR existante, état de l'issue Plane.

| Phase | Quand | Instructions |
|---|---|---|
| `start` | « crée l'issue », « je prends la tâche » | `phases/start.md` (crée l'issue si elle n'existe pas) |
| `mr` | « j'ai fini », « commit », « push », « fais la MR » | `phases/mr.md` |
| `status` | « où j'en suis » | lire branche, changements, MR, issue Plane ; dire en une ligne la phase suivante |

Charger les outils MCP de la phase (liste dans `references/mcp-tools.md`) en un seul ToolSearch, puis exécuter la phase d'un trait. S'arrêter seulement pour une vraie ambiguïté ou une validation due à l'utilisateur (message de commit).

## Prérequis

Vérifier au début de chaque phase. Manquant → arrêter, donner l'étape de connexion, ne rien faire d'autre.

- **MCP Plane** et **MCP GitLab** connectés et authentifiés (`/mcp`). Aucune clé d'API, jamais `glab`.
- Intégration GitLab active sur le projet Plane (le label `gitlab` crée l'issue GitLab).
- Repo avec un remote GitLab ; `git` local pour fetch, switch, push.
- Optionnel : MCP Playwright, pour les captures de `mr`.

## Outils MCP

Les outils sont nommés sans préfixe dans le skill (`get_me`, `save_merge_request`…). Le préfixe dépend de la connexion : connecteurs claude.ai → `mcp__claude_ai_Plane__` / `mcp__claude_ai_Gitlab__` ; serveur ajouté à la main → `mcp__<nom du serveur>__`. Ne jamais supposer le préfixe : chercher chaque outil par son nom (ToolSearch, ex. `select:` après un `+plane get_me`) et utiliser le préfixe trouvé. Aucun outil Plane ou GitLab trouvé → prérequis manquant (voir ci-dessus).

`allowed-tools` ci-dessus ne couvre que les connecteurs claude.ai. Autre préfixe → des demandes de permission apparaissent ; les éviter en ajoutant les mêmes outils avec le bon préfixe dans `.claude/settings.json` (`permissions.allow`). Jamais de joker sur tout le serveur : il autoriserait aussi les outils de suppression.

## Config

`config.yml` dans ce dossier, versionnée avec le projet. Lire au début de chaque phase. Absente → setup automatique. Détail : `references/config.md`. Erreurs courantes : `references/troubleshooting.md`.

## États Plane

Au début de chaque phase, `list_states` du projet de l'issue : les noms réels font foi, rien en dur. Rôles utilisés : « Backlog », « Sprint Backlog », « En cours », « Test ». Pour chaque rôle, chercher le nom (casse et accents ignorés), sinon l'état du même groupe Plane (`backlog`, `unstarted`, `started`) ; plusieurs candidats ou aucun → demander lequel prendre. Ne jamais poser un état du groupe `completed` ou `cancelled`.

## Règles

Source unique des règles : les phases et le README s'y réfèrent sans les redéfinir.

- Commit : toujours proposer le message et attendre la validation de l'utilisateur ; jamais sans. Jamais de push force, de merge de MR, de suppression d'issue.
- Plane après la MR : le skill passe seulement l'état en « Test » ; temps passé et captures sont à la main par l'utilisateur.
- Jamais d'issue Plane en "Terminé" ni "Annulé".
- Jamais de création de label GitLab : seuls les labels existants sont utilisés.
- Toute MR cible la branche par défaut du repo GitLab (`get_project` → `default_branch`), sans exception.
- Les lignes `REF <n>` et `v<x.y.z>` dans la description d'une issue Plane sont des références internes **sans lien** avec les numéros GitLab (`#42`) ni Plane (`PROJ-123`). Ne jamais s'en servir pour retrouver une issue, nommer une branche ou écrire `Closes`. Ne jamais les recopier dans la MR. Les laisser telles quelles dans la description.
- Date du jour : toujours `date +%F`, jamais une supposition.
- Action sortante (Plane, MR) : résumer ce qui est envoyé, sans reconfirmer.
- Texte publié (MR) dans `defaults.language` (défaut : français), prose normale.
