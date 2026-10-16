# Guide de publication sur GitHub

À exécuter sur **ta machine** (pas dans une VM du lab), dans le dossier
`zero-trust-lab/` une fois que tu y as copié tes propres fichiers en plus
de ceux fournis ici (screenshots dans `docs/screenshots/`, export réel du
realm Keycloak, etc.).

## 1. Dernière relecture avant publication

```bash
cd zero-trust-lab
grep -r -i -E "password|secret|BEGIN (RSA|PRIVATE)" --include="*.yml" --include="*.yaml" --include="*.json" --include="*.tex" .
```

Relis chaque résultat : seuls les placeholders (`CHANGE_ME`, `*_REDACTED`)
et les mots de passe volontairement faibles des comptes de test
(`Jdupont123`, etc., documentés comme tels) doivent apparaître.

## 2. Initialiser le dépôt local

```bash
git init
git add .
git status   # vérifie qu'aucun .env, .key ou fichier non voulu n'est listé
git commit -m "Initial commit: Zero Trust Lab (Phase A + Phase B steps 1-6)"
```

## 3. Créer le dépôt sur GitHub

**Option A — via le site web**
1. github.com → New repository → nom `zero-trust-lab` → **Public** →
   ne coche ni README ni .gitignore ni licence (déjà présents ici).
2. Crée le dépôt, puis copie l'URL affichée (HTTPS ou SSH).

**Option B — via GitHub CLI** (si `gh` est installé et authentifié)

```bash
gh repo create zero-trust-lab --public --source=. --remote=origin
```

## 4. Pousser le code

```bash
git branch -M main
git remote add origin <URL_DE_TON_DEPOT>   # si tu as utilisé l'option A
git push -u origin main
```

## 5. Après publication

- Ajoute une description courte et des topics (`zero-trust`, `network-security`,
  `keycloak`, `opnsense`) sur la page GitHub du dépôt pour la découvrabilité.
- Mets le lien du dépôt sur ton CV / LinkedIn une fois le README relu une
  dernière fois en ligne (le rendu markdown GitHub peut différer légèrement).
- Pour les prochains steps (7 à 14), le flux est le même : relecture des
  secrets → `git add` → `git commit` → `git push`. Pas besoin de refaire
  `git init`.

## Rappel important

Une fois un secret poussé sur GitHub, même si tu le supprimes dans un commit
suivant, **il reste visible dans l'historique git** tant que l'historique
n'est pas réécrit (`git filter-repo` ou équivalent, en dernier recours et
avec `--force` sur le push). Mieux vaut vérifier avant chaque commit que
de corriger après coup.
