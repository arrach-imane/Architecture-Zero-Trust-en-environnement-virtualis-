# Tests de validation

Chaque test : action → résultat attendu → interprétation.

## Segmentation et isolation

| Test | Action | Attendu | Interprétation |
|---|---|---|---|
| Isolation MONITORING | Depuis Server-Monitoring : SSH/TCP vers IAM et APPS | Timeout | Flux unidirectionnel prouvé |
| Contournement par IP | `curl http://10.10.3.30/` | HTTP 400 | Nextcloud refuse l'accès hors domaine de confiance |
| Port Keycloak | Accès direct 10.10.2.10:8080 | Refus | Port retiré, Traefik seul point d'entrée |

### Incident utile : règle oubliée

Le test négatif depuis MONITORING a révélé une règle `any→any` TCP non documentée et sans journalisation sur OPT5 : MONITORING atteignait IAM/APPS en SSH. La règle a été désactivée, les tests sortants ont tous expiré (timeout), puis elle a été supprimée. Leçon : un audit des règles ne remplace pas un test négatif.

## Authentification

| Test | Action | Attendu |
|---|---|---|
| Sans session | `curl -I https://whoami.zerotrust.lab` | 302 vers `auth.zerotrust.lab` (401 côté API, visible dans les logs Traefik) |
| MFA | Connexion Authelia | Mot de passe puis code TOTP exigés |
| Chaîne SSO | Accès à Nextcloud après Authelia | Redirection Keycloak puis tableau de bord |
| URI de redirection | Redirect URI non enregistrée | Keycloak : `Invalid parameter: redirect_uri` |

## Supervision

4 agents Wazuh actifs (001 server-iam, 002 server-nextcloud, 003 server-db, 004 kali-attacker) visibles dans le dashboard.

## Rejeu de la Phase A (Step 12)

| Attaque | Phase A | Phase B |
|---|---|---|
| Scan Nmap | Tous services visibles | À compléter |
| Force brute SSH | Succès en 2 min | À compléter |
| Accès direct DB | Succès | À compléter |
| Interception HTTP | Succès | À compléter |
