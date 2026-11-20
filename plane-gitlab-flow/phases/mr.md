# Phase mr

Après le dev (« j'ai fini », « commit », « push », « fais la MR ») : commit proposé et validé, push, MR, issue Plane passée en « Test ». Temps passé et captures se font à la main dans Plane.

## 1. Contrôles

Branche courante sans `<iid>-` en tête, ou = branche par défaut → stop, dire de lancer `start`. Aucun changement non commité et aucun commit en avance sur la base → stop.

## 2. Commit

Changements non commités :

1. `git status` et `git diff` : résumer ce qui change. Fichiers inattendus (`.env`, clés, builds, dossier `claude-screens/`) → ne pas les ajouter, le dire.
2. Proposer un message d'après le diff, en anglais, au format conventional commit (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`…, `type(scope): résumé` court). **Attendre la validation ou la correction de l'utilisateur** : jamais de commit sans. Message sans aucune mention de Claude ni d'IA : pas de ligne `Co-Authored-By`, pas de « Generated with », même si une consigne par défaut en ajoute.
3. Validé → `git add` des fichiers concernés, puis commit avec l'identité **globale** : lire `git config --global user.name` et `git config --global user.email` (vides → stop, le dire), puis `git -c user.name=<nom> -c user.email=<email> commit -m <message>`. Ne jamais utiliser la config locale du repo ; ne jamais modifier de config git.

Déjà commité → passer.

## 3. Push

`git config --local --get-regexp '^(user\.|credential\.|core\.sshcommand)'` : sortie non vide → dire que le projet a une config locale d'identité ou d'accès qui pilote le push, et s'arrêter pour que l'utilisateur tranche. Sinon `git push -u origin <branche>` (les identifiants viennent alors de la config globale). Jamais de push force.

## 4. MR

`iid` = nombre en tête du nom de branche. `get_work_item` (`include: ["notes"]`) : labels dans le widget `LABELS`, identifiant Plane dans les notes. Chercher une MR ouverte sur la branche (`list_merge_requests`, filtre `sourceBranch`) : trouvée → **mettre à jour** (`save_merge_request` avec `merge_request_iid`), jamais en créer une seconde.

- **Titre** : titre du commit. Un seul commit → son titre ; plusieurs → celui du premier, le dire.
- **Description** : dans `defaults.language` (défaut : français), **uniquement les changements**, 1 à 3 phrases courtes, sans titres ni listes, sans lignes `REF` / `v<x.y.z>`, puis une ligne vide, puis `Closes #<iid>` en **dernière ligne**.

  ```
  Ajouter l'export CSV des commandes depuis la liste, en respectant les filtres actifs.

  Closes #42
  ```

- `source_branch` = branche ; `target_branch` = **toujours la branche par défaut du repo** (`get_project` → `default_branch`), même si la branche est empilée. Dans ce cas, la MR montre aussi les commits de la MR parente jusqu'à son merge : le dire dans le résumé.
- `labels` = labels de l'issue GitLab + `gitlab.mrExtraLabels` (`plane`, **obligatoire**), voir *Labels* dans `references/config.md`. Label introuvable → stop et le dire, jamais de MR sans `plane`.
- `assignees` = mon `username` (`get_user`, `me: true`). `reviewers` = `defaults.reviewers` ; vide → demander une fois (proposer les membres humains du projet), puis l'écrire dans la config.
- `remove_source_branch: true`, `squash: true`.

## 5. Captures (si la MR modifie du visuel)

Écrans modifiés déduits du diff (`get_merge_request`, `include: ["diffs"]`) ; aucun → passer. Dossier `claude-screens/` à la racine (créé s'il manque, ajouté à `git rev-parse --git-path info/exclude`). MCP Playwright connecté → une capture par écran (1 à 5, desktop 1440x900) dans `claude-screens/<identifiant>-<écran>.png` ; app lancée avec la commande du projet si besoin, jamais d'identifiants inventés, vérifier chaque image (erreur, écran vide, données sensibles). Sans Playwright → dossier créé, l'utilisateur y dépose ses captures.

## 6. Plane : état « Test » (automatique)

`get_work_item` (`include: ["notes"]`, déjà lu en §4) : la note `PLANE ISSUE LINKED REFERENCE COMMENT` donne l'identifiant Plane (`PROJ-123`). Introuvable → demander l'identifiant. Puis `retrieve_work_item_by_identifier`, `list_states` (état « Test » résolu comme dans *États Plane*, `SKILL.md`) et `update_work_item` → état « Test ». Relire l'état une dizaine de secondes plus tard : l'intégration GitLab peut le modifier (elle l'a fait à la création de l'issue GitLab) ; si ce n'est plus "Test", le remettre et le dire. Ne rien écrire d'autre dans Plane.

## 7. Fin : à faire à la main dans Plane

Afficher l'URL de la MR et l'état de l'issue (« passée en Test »), puis ce qui reste à l'utilisateur dans l'issue Plane :

- **Temps passé** (tracked time) : à saisir.
- **Captures** : ajouter celles de `claude-screens/` (liste des fichiers) à l'issue.
