# Tests de validation

Chaque test suit le même format : **action → résultat attendu → résultat observé → interprétation**. Les sorties ci-dessous sont celles relevées dans le lab.

## 1. Segmentation et isolation

### T1 — Le port 8080 de Keycloak n'est plus joignable

Depuis Client-Debian (MGMT), avant durcissement le port d'administration directe était exposé. Il a été retiré du compose : Traefik est l'unique point d'entrée.

```console
debian-user@client-debian:~$ nc -zv -w3 10.10.2.10 8080
whoami.zerotrust.lab [10.10.2.10] 8080 (http-alt) : Connection timed out
```

**Interprétation** : `timed out` (et non `refused`) indique que le paquet est abandonné par le pare-feu, sans réponse : le service n'est même pas confirmé comme existant.

### T2 — Nextcloud refuse l'accès par adresse IP

Depuis Server-IAM, requête directe vers Nextcloud par son IP puis par son nom de domaine :

```console
$ curl -s -o /dev/null -w "%{http_code}\n" --max-time 15 http://10.10.3.30/
400
$ curl -ks -o /dev/null -w "%{http_code}\n" --max-time 15 --resolve nextcloud.zerotrust.lab:... http://nextcloud.zerotrust.lab/
302
```

**Interprétation** : l'accès par IP est rejeté (`400`, domaine non fiable) ; seul le nom `nextcloud.zerotrust.lab` est accepté (`302` vers l'authentification). Un attaquant qui atteindrait l'IP ne peut pas contourner Traefik/Authelia.

### T3 — Flux Traefik → Nextcloud

```console
$ nc -zv -w4 10.10.3.30 80
(UNKNOWN) [10.10.3.30] 80 (http) open
$ nc -zv -w4 10.10.3.30 443
(UNKNOWN) [10.10.3.30] 443 (https) : Connection refused
```

**Interprétation** : seul HTTP/80 est exposé par Nextcloud vers le segment IAM, le TLS est terminé par Traefik.

### T4 — Isolation du segment MONITORING (test négatif)

Depuis Server-Monitoring, tentatives de connexion sortantes vers IAM et APPS (SSH, TCP) : **toutes expirent**. Le flux Wazuh est strictement unidirectionnel (agents → manager).

#### Incident révélé par ce test : règle oubliée

Le test a d'abord *réussi là où il aurait dû échouer* : une règle `any→any` TCP non documentée, sans journalisation, sur OPT5 permettait à MONITORING d'atteindre IAM/APPS en SSH. La règle a été désactivée, les tests sortants ont tous expiré, puis la règle a été supprimée.

**Leçon** : un audit visuel des règles ne remplace pas un test négatif exécuté.

## 2. Authentification

| Test | Action | Attendu | Observé |
|---|---|---|---|
| T5 Sans session | `curl -I https://whoami.zerotrust.lab` | Redirection vers Authelia | `302` vers `auth.zerotrust.lab` ; `401` visibles dans les logs Traefik côté forward-auth |
| T6 MFA | Connexion Authelia | Mot de passe puis TOTP exigés | Code à usage unique demandé après le mot de passe |
| T7 Chaîne SSO | Accès à Nextcloud après Authelia | Redirection Keycloak puis tableau de bord | Tableau de bord Nextcloud affiché (utilisateur `Test User`) |
| T8 URI de redirection | Redirect URI non enregistrée | Rejet Keycloak | `Invalid parameter: redirect_uri` |

## 3. Supervision

| Test | Attendu | Observé |
|---|---|---|
| T9 Agents Wazuh | 4 agents actifs | 001 server-iam, 002 server-nextcloud, 003 server-db, 004 kali-attacker |

## 4. Rejeu de la Phase A (Step 12)

| Attaque | Phase A (réseau plat) | Phase B (Zero Trust) |
|---|---|---|
| Scan Nmap | Tous services visibles | À compléter |
| Force brute SSH | Succès en 2 min | À compléter |
| Accès direct DB | Succès | À compléter |
| Interception HTTP | Succès | À compléter |
