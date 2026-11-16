# Dépannage

| Symptôme | Cause probable | Action |
|---|---|---|
| Outils Plane ou GitLab introuvables | MCP non connecté | `/mcp`, connecter Plane et GitLab, relancer la phase |
| MCP "Needs authentication" | session expirée | `/mcp`, choisir le serveur, s'authentifier |
| Issue GitLab introuvable après `start` (`external_id` vide au bout de 2 min) | sync lente, intégration GitLab inactive sur le projet Plane, ou label `gitlab` absent | attendre, vérifier l'intégration et le label, relancer `start` |
| Branche `<numéro>-*` déjà sur `origin` | `start` déjà lancé | reprendre la branche existante |
| `mr` s'arrête | branche sans `<iid>-` en tête (ou branche par défaut), ou rien à commiter ni à pousser | lancer `start`, ou faire un changement |
| MR refusée : label introuvable | label absent de GitLab | corriger `labels` dans `config.yml`, ou créer le label dans GitLab |
| Mauvaise base de branche | ordre de création des MR différent de l'ordre d'empilement | corriger la base annoncée par le skill |
| État Plane introuvable | colonne renommée | le skill demande lequel prendre |
| Pas de captures dans `claude-screens/` | MCP Playwright absent, ou app non lancée | connecter Playwright, ou déposer les captures à la main dans le dossier |
| Dépôt sans commit | rien dont partir | faire le premier commit sur la branche par défaut, le pousser |
