# Skill : review-security

Audit de sécurité ciblé sur les fichiers modifiés dans la session ou la branche courante.

## Étapes à exécuter

### 1. Identifier le périmètre
- `git diff --name-only $(git merge-base HEAD develop)..HEAD` — fichiers modifiés depuis develop
- Si aucun résultat, utiliser `git diff --name-only HEAD~1..HEAD`
- Lire chaque fichier modifié

### 2. Appliquer la checklist OWASP Top 10

Pour chaque fichier, vérifier :

| Risque | Ce qu'on cherche |
|--------|-----------------|
| **Injection** | Concaténation de user input dans SQL, shell, LDAP — remplacer par requêtes paramétrées |
| **Auth cassée** | Tokens hardcodés, pas de hash sur mots de passe, JWT sans expiry |
| **Exposition de données** | Logs contenant PII, tokens, mots de passe |
| **XSS** | User input rendu en HTML sans échappement ni CSP |
| **CSRF** | Requêtes state-changing sans token CSRF ni SameSite |
| **Mauvaise config** | Debug mode actif, headers de sécurité manquants (HSTS, X-Frame-Options) |
| **Dépendances vulnérables** | Libs connues comme vulnérables — suggérer `npm audit` / `pip audit` |
| **Secrets** | Clés API, tokens, credentials dans le code ou les configs committés |
| **Validation d'input** | Inputs non validés à la frontière (type, longueur, format) |
| **Droits excessifs** | Permissions trop larges (DB, API keys, IAM) |

### 3. Produire un rapport structuré

Format de sortie :

```
## Rapport de sécurité — <date>

### ✅ Aucun problème détecté
- <fichier> — OK

### ⚠️ Points d'attention (à corriger avant merge)
- <fichier>:<ligne> — <risque OWASP> — <description> — <fix suggéré>

### ❌ Blocants (ne pas merger sans correction)
- <fichier>:<ligne> — <risque OWASP> — <description> — <fix suggéré>

### Recommandations complémentaires
- <suggestion non bloquante>
```

### 4. Proposer les corrections

Pour chaque point ⚠️ ou ❌, proposer le fix exact au format "Domaine : Sécurité / Proposition : ... / Pourquoi : ... / Impact : ... → Valider ?"

---

> Toute correction identifiée comme bloquante doit être validée par l'utilisateur avant d'être appliquée.
