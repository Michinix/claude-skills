# Phase start

Argument : issue Plane (URL, `PROJ-45`, ou titre). Absent → demander.

## 0. Issue inexistante

Tâche décrite sans issue existante, ou création demandée : la créer d'abord. Demande de création seule → s'arrêter après la création ; « crée et prends-la » → continuer en §1.

- **Titre** : user story de 20 mots max (15-20 visés) décrivant ce qu'on veut faire, forme « En tant que <rôle>, je veux <action> » (+ « afin de <but> » si ça tient), avec les mots de l'utilisateur, sans numéro. **Description** : texte de l'utilisateur gardé tel quel, y compris ses lignes `REF <n>` / `v<x.y.z>` (jamais inventées) ; sinon 2-6 lignes (contexte, attendu, étapes pour un bug).
- **Type** toujours `Issue` (`list_work_item_types`) ; bug, feature, etc. = labels. **Priorité** en minuscules (`urgent`, `high`, `medium`, `low`, `none`) : celle dite, sinon `medium`, `urgent` seulement si dit. **État** « Backlog » (ou « Sprint Backlog » si « dans le sprint »), résolu comme dans *États Plane* (`SKILL.md`). Labels et estimation : comme en §1.
- Doublons : `search_work_items`, résultats du même projet (préfixe `PROJ-`) ; titre quasi identique → la montrer et demander.
- `create_work_item` en une fois (`name`, `description_html`, `type_id`, `priority`, `state`, `labels`, `estimate_point`). Plusieurs issues demandées → toutes, récap en liste. Afficher identifiant et lien.

## 1. Issue Plane

Lire l'issue (`retrieve_work_item_by_identifier`). Puis, par appels séparés :

- **Assignee** : moi (`get_me`, `manage_work_item_assignee`).
  - Personne, ou déjà moi → m'ajouter.
  - Issue que `create` vient de créer, assignée par défaut par le projet → remplacer sans demander.
  - Quelqu'un d'autre → demander (« `<nom>` est assigné à cette issue, je la reprends à sa place ? »). Oui → le retirer et m'ajouter. Non → stop.
- **Dates** début et fin = `date +%F` (`update_work_item`). L'**état** « En cours » ne se pose pas ici : voir §2.
- **Estimation** : si vide, l'estimer sans demander (Fibonacci : 1, 2, 3, 5, 8, 13, 21). Sources : titre, description, module, cycle ; contexte projet (`CLAUDE.md`, stack) ; fichiers probablement touchés (les lire) ; issues voisines similaires comme étalon. Critères : taille, fichiers et couches touchés, risque, inconnues. Annoncer `Estimation : 5 (raison)`. Déjà remplie → la garder. Écrire avec `estimate_point` (UUID du point).
- **Labels** : garder les existants. Aucun label métier → en choisir 1 (2 si évident) parmi `labels` (config), d'après `when`. Un `manage_work_item_label` par label. **Le label `gitlab` se pose en dernier**, après tous les autres champs (labels, estimation, assignee, état, dates) : il déclenche la sync, et l'issue GitLab copie ce qui existe à ce moment-là.

Ne rien redemander de déjà renseigné. Ne pas toucher à la priorité.

## 2. Issue GitLab

Le label `gitlab` déclenche la sync. Attendre : relire l'issue Plane (`fields=external_id,external_source`) toutes les 10 s, 2 min max, jusqu'à ce que `external_source` soit renseigné (ex. `GITLAB_ENTERPRISE`).

`external_id = gl_<id projet>_<iid>` → `iid` = numéro de l'issue GitLab. Vérifier avec `get_work_item` : le titre doit être celui de l'issue Plane. Jamais de recherche par titre, jamais de numéro deviné ou calculé. Rien après 2 min → le dire, stop (voir `troubleshooting.md`). Ne pas créer l'issue GitLab à la main.

**État « En cours » : à poser maintenant, après la sync.** Le bot de sync remet l'issue Plane dans l'état « unstarted » (ex. « Sprint Backlog ») quand il crée l'issue GitLab. `update_work_item` → état « En cours » (résolu comme dans *États Plane*), puis relire l'état quelques secondes plus tard pour vérifier qu'il tient.

## 3. Base de la branche

Les branches s'empilent **seulement avec mes propres MR** ; ignorer celles des autres.

1. Lister mes MR ouvertes (`scope: created_by_me`, `state: opened`).
2. Aucune → base = `gitlab.stagingBranch`. Dépôt sans commit sur `origin` → stop, demander un premier commit sur la branche par défaut.
3. Sinon base = branche source de **ma MR ouverte la plus récemment créée**.

Annoncer base et raison, laisser corriger. La base ne sert qu'à créer la branche : la MR vise toujours la branche par défaut (`phases/mr.md`).

## 4. Branche

Nom = celui que GitLab génère pour l'issue : `<iid>-<titre>`.

- Titre en minuscules, accents retirés, tout caractère hors `a-z0-9` devient `-`, tirets multiples fusionnés.
- Ajouter les mots un par un tant que le nom reste ≤ 100 caractères ; un mot qui dépasse est retiré en entier.
- Exemple : `42-en-tant-qu-utilisateur-je-veux-exporter-les-commandes-en-csv`.

Branche déjà sur `origin` (`list_branches`, `search: ^<iid>-`) → ne pas recréer, le dire, demander.

Créer côté GitLab : `add_branch` (`branch`, `ref` = base). Puis : `git fetch origin && git switch <branche>`.

Afficher : lien Plane, lien GitLab, branche, base. Laisser l'utilisateur dev.
