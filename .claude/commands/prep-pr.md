# Skill : prep-pr

Génère le corps complet d'une Pull Request pour la branche courante, prêt à soumettre.

## Étapes à exécuter

### 1. Collecter le contexte de la branche
- `git log --oneline $(git merge-base HEAD develop)..HEAD` — liste des commits
- `git diff $(git merge-base HEAD develop)..HEAD --stat` — fichiers modifiés
- Lire la phase doc correspondante dans `docs/phases/` si elle existe

### 2. Lire l'état de la documentation
- Vérifier que `docs/phases/phase-X.Y-*.md` existe et que le status est à jour
- Vérifier que `docs/phases/phase-X.Y-test-plan.md` est complété
- Vérifier que `docs/phases/phase-X.Y-user-doc.md` est complété
- Vérifier que `CHANGELOG.md` a au moins une entrée pour cette phase

### 3. Générer le corps de PR

Utiliser exactement le template suivant, rempli à partir du contexte collecté :

```markdown
## Summary
<!-- Ce qui a été construit, pourquoi, comment — 3 à 5 bullets -->

## Phase Documentation
- [ ] `docs/phases/phase-X.Y-*.md` créé et à jour
- [ ] `docs/phases/phase-X.Y-test-plan.md` créé, tous les cas exécutés
- [ ] `docs/phases/phase-X.Y-user-doc.md` créé et couvre toutes les features
- [ ] `docs/RECAP.md` mis à jour
- Link: docs/phases/phase-X.Y-<nom>.md

## Changes
- [ ] <fichier modifié> — <raison>
...

## Tests
- [ ] Tests unitaires écrits et passants
- [ ] Tests d'intégration écrits et passants
- [ ] Cas limites couverts
- [ ] Output de test / coverage attaché

## Security
- [ ] Aucun secret commité
- [ ] Inputs validés aux frontières
- [ ] `npm audit` / équivalent lancé — aucun HIGH/CRITICAL
- [ ] Auto-review OWASP Top 10 effectuée

## Definition of Done
- [ ] Feature fonctionne comme spécifiée
- [ ] Aucune régression sur les tests existants
- [ ] Code reviewé pour l'élégance
- [ ] `docs/phases/phase-X.Y-*.md` finalisé (status: Done)
- [ ] `CHANGELOG.md` mis à jour
- [ ] `README.md` mis à jour si nécessaire
- [ ] `tasks/todo.md` mis à jour
- [ ] `docs/lessons.md` mis à jour si une erreur a été corrigée
```

### 4. Signaler les éléments manquants

Lister clairement ce qui n'est pas encore fait avant que la PR puisse être ouverte. Proposer de le compléter immédiatement si possible.

---

> Ne pas créer la PR via `gh pr create` sans validation explicite de l'utilisateur sur le corps généré.
