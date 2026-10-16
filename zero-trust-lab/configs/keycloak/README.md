# Export du realm Keycloak

Place ici l'export JSON de ton realm `zerotrust` (Keycloak → Realm settings →
Action → Partial export, ou `kc.sh export`).

**Avant de committer cet export**, vérifie et retire :
- tout `secret` de client OIDC (champ `"secret": "..."` sous
  `clients[].secret`) — régénère-le côté Keycloak après l'export si tu dois
  en partager un exemple fonctionnel ;
- les éventuels hashs de mot de passe utilisateurs si tu exportes aussi les
  utilisateurs (`users` dans le JSON) — en général on exporte uniquement la
  configuration du realm, sans les comptes ;
- toute clé privée stockée dans `realm-export.json` (clés de signature).

Une commande utile pour vérifier rapidement avant de pousser :

```bash
grep -i -E '"secret"|"privateKey"|BEGIN (RSA|PRIVATE)' realm-export.json
```

Si cette commande renvoie des résultats, ne commite pas le fichier tel quel.
