# Skill : new-feature

Initialise une nouvelle feature de A à Z avant d'écrire la moindre ligne de code.

## Feature à créer
$ARGUMENTS

## Étapes à exécuter dans l'ordre

### 1. Lire le contexte existant
- Lire `tasks/todo.md` pour connaître l'état courant
- Lire `docs/phases/` pour identifier le numéro de phase suivant
- Lire `CHANGELOG.md` pour s'aligner sur la version courante

### 2. Identifier le numéro de phase
- Déterminer le prochain numéro X.Y disponible dans `docs/phases/`
- Si aucune phase n'existe encore, démarrer à 1.0

### 3. Créer les fichiers de documentation de phase

Créer les trois fichiers suivants (status: Draft) :

- `docs/phases/phase-X.Y-<nom-court>.md` — Phase doc principale
- `docs/phases/phase-X.Y-test-plan.md` — Cahier de tests
- `docs/phases/phase-X.Y-user-doc.md` — Documentation utilisateur

Utiliser exactement les structures définies dans CLAUDE.md.

### 4. Écrire le plan dans tasks/todo.md

Ajouter une section pour cette feature avec :
- [ ] Tâches de développement (granulaires, actionnables)
- [ ] Tâches de tests
- [ ] Tâches de documentation

### 5. Proposer les décisions techniques

Pour chaque domaine technique impliqué (architecture, DB, sécurité, CI, etc.), formuler une proposition au format :

```
Domaine : <domaine>
Proposition : <ce que je propose>
Pourquoi : <justification>
Impact : <risques / changements>
→ Valider pour continuer ?
```

### 6. Proposer le nom de branche

```
feature/phase-X.Y-<nom-court>
```

Attendre la validation de l'utilisateur avant de créer la branche ou d'écrire du code.

---

> Ne jamais commencer l'implémentation sans validation explicite de l'utilisateur sur le plan et les décisions techniques.
