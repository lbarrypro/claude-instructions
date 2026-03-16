# Skill : debt-review

Inventaire exhaustif de la dette technique dans la codebase courante.

## Étapes à exécuter

### 1. Scanner les marqueurs de dette

Chercher dans tous les fichiers source (hors `node_modules`, `.git`, `dist`, `build`) :
- `TODO`
- `FIXME`
- `HACK`
- `TEMP`
- `XXX`
- `WORKAROUND`
- `@deprecated`
- `// eslint-disable`
- `any` (TypeScript — signal de type casting forcé)

### 2. Classifier chaque item

| Priorité | Critère |
|----------|---------|
| 🔴 **Bloquant** | Bloque 1+ feature, risque de sécurité, ou migration en attente |
| 🟡 **Important** | Ralentit le dev, mauvaise lisibilité, duplicata avéré |
| 🟢 **Mineur** | Nettoyage cosmétique, renommage, commentaire obsolète |

### 3. Identifier la dette structurelle

Au-delà des marqueurs, détecter :
- Fonctions > 50 lignes sans découpage
- Fichiers > 300 lignes sans justification
- Duplication de logique métier (DRY violation)
- Dépendances non utilisées (imports morts)
- Fichiers de migration modifiés après merge (violation CLAUDE.md §13)

### 4. Produire le rapport

Format de sortie :

```
## Rapport de dette technique — <date>

### 🔴 Bloquant (<N> items)
- `<fichier>:<ligne>` — <marqueur> — <description> — <impact> — Ticket suggéré : #TODO

### 🟡 Important (<N> items)
- `<fichier>:<ligne>` — <marqueur> — <description>

### 🟢 Mineur (<N> items)
- `<fichier>:<ligne>` — <marqueur> — <description>

### Dette structurelle
- <description du problème> — <fichier(s) concerné(s)>

### Recommandation de priorisation
Items à traiter avant la prochaine phase : <liste>
Items à planifier dans un chore/deps : <liste>
```

### 5. Proposer un plan d'action

Si des items bloquants existent : proposer de les traiter immédiatement dans un ticket `chore/debt-<description>`.

---

> Ne rien modifier sans validation de l'utilisateur. Ce skill est en lecture seule.
