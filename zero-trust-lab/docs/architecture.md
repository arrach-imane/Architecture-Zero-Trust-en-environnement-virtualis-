# Architecture

## Segments et machines

| Segment | Interface OPNsense | Sous-réseau | Machine | IP |
|---|---|---|---|---|
| MGMT | LAN (em1) | 10.10.1.0/24 | Client-Debian | 10.10.1.50 |
| IAM/DMZ | OPT1 (em2) | 10.10.2.0/24 | Server-IAM | 10.10.2.10 |
| APPS | OPT2 (em3) | 10.10.3.0/24 | Server-Nextcloud | 10.10.3.30 |
| DB | OPT3 (em4) | 10.10.4.0/24 | Server-DB | 10.10.4.20 |
| ATTACKER | OPT4 (em5) | 10.10.5.0/24 | Kali | 10.10.5.50 |
| MONITORING | OPT5 (em6) | 10.10.6.0/24 | Server-Monitoring | 10.10.6.10 |

## Nommage et TLS

Tous les services sont publiés sous `*.zerotrust.lab` (`whoami`, `auth`, `keycloak`, `nextcloud`) et résolus vers 10.10.2.10 : **Traefik est le point d'entrée unique**. Certificat auto-signé `CN=*.zerotrust.lab` avec SAN.

## Double couche d'authentification

1. **Authelia** (forward-auth via Traefik) : politique `deny` par défaut, `two_factor` (mot de passe + TOTP) pour whoami et Nextcloud.
2. **Keycloak** (realm `zerotrust`) : SSO OIDC pour Nextcloud via l'application `user_oidc` (mapping `preferred_username`, URI de redirection exactes).

Flux : client → Traefik → Authelia (MFA) → Nextcloud → redirection Keycloak (OIDC) → session Nextcloud.

## Supervision

Wazuh 4.14.9 (single-node Docker) sur Server-Monitoring avec 4 agents (IAM, Nextcloud, DB, Kali). Les agents se connectent **vers** le manager (1514/1515) ; le segment MONITORING n'a aucune règle sortante (flux strictement unidirectionnel). Voir [wazuh.md](wazuh.md).
