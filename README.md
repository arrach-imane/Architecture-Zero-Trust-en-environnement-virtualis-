# Zero Trust Lab

Architecture Zero Trust complète bâtie dans un environnement virtualisé
(VirtualBox) : segmentation réseau, gestion centralisée des identités,
reverse proxy avec authentification multifacteur et supervision —
démontrée par une chaîne d'attaque réelle rejouée avant et après
sécurisation.

## Contexte

Projet réalisé dans le but de comprendre le principe zero Trust et de l'appliquer

L'objectif n'est pas seulement de déployer des briques Zero Trust, mais de
**prouver** leur efficacité : la Phase A construit volontairement une
architecture réseau plate et vulnérable, puis une chaîne d'attaque complète
(reconnaissance → brute force → exfiltration) est menée avec succès contre
elle. La Phase B reconstruit la même infrastructure selon les principes
Zero Trust, et la même chaîne d'attaque est rejouée pour mesurer
concrètement la réduction de la surface d'exposition.

## Architecture

**Phase A — réseau plat (baseline)**

Un unique sous-réseau `192.168.1.0/24`, sans segmentation ni règle de
filtrage, hébergeant OPNsense, un serveur MariaDB, un serveur Nextcloud et
un poste attaquant Kali Linux.

**Phase B — architecture Zero Trust**

Le réseau est redécoupé en six segments isolés, reliés uniquement via
OPNsense qui applique une politique de moindre privilège :

| Segment     | Sous-réseau    | Machine (IP)                  | Rôle                                         |
|-------------|----------------|-------------------------------|-----------------------------------------------|
| MGMT        | 10.10.1.0/24   | Client-Debian (10.10.1.50)    | Administration du lab                        |
| IAM / DMZ   | 10.10.2.0/24   | Server-IAM (10.10.2.10)       | Identité (Keycloak, Authelia, Traefik)       |
| APPS        | 10.10.3.0/24   | Server-Nextcloud (10.10.3.30) | Applications (Nextcloud)                     |
| DB          | 10.10.4.0/24   | Server-DB (10.10.4.20)        | Base de données (MariaDB)                    |
| ATTACKER    | 10.10.5.0/24   | Kali (10.10.5.50)             | Poste d'attaque                              |
| MONITORING  | 10.10.6.0/24   | Server-Monitoring (10.10.6.10)| Supervision (Wazuh)                          |

Chaque segment ne communique qu'avec les segments strictement nécessaires
à sa fonction (ex. le segment DB n'a aucune règle sortante : il ne fait que
répondre aux requêtes déjà autorisées en entrée depuis APPS).

Les services sont publiés sous `*.zerotrust.lab` derrière **Traefik, point
d'entrée unique**, avec une double couche d'authentification : Authelia
(mot de passe + TOTP) puis SSO Keycloak (OIDC) pour Nextcloud. Détails dans
[zero-trust-lab/docs/architecture.md](zero-trust-lab/docs/architecture.md).

## Démarche

1. **Step 1-4** — Phase A : mise en place du réseau plat, de Server-DB et
   Server-Nextcloud, et chaîne d'attaque de référence (scan Nmap → OSINT/
   CUPP → brute force SSH Hydra → extraction des identifiants → accès
   direct à la base de données → interception de trafic HTTP en clair).
2. **Step 5-6** — Phase B : segmentation réseau en six segments OPNsense et
   définition des règles de pare-feu selon le principe de moindre
   privilège, avec vérification positive/négative par segment.
3. **Step 7-9** ✅ — Brique IAM : Keycloak (identités, realm `zerotrust`),
   Traefik (reverse proxy), Authelia (MFA) et SSO OIDC vers Nextcloud.
4. **Step 10** ✅ — Supervision Wazuh 4.14.9 sur le segment MONITORING,
   4 agents, flux strictement unidirectionnel ([docs/wazuh.md](zero-trust-lab/docs/wazuh.md)).
5. **Step 11** ✅ — Durcissement : TOTP obligatoire, port Keycloak 8080
   retiré, URI de redirection exactes, secrets Authelia régénérés, compte
   admin Keycloak permanent.
6. **Step 12** *(en cours)* — Rejeu de la chaîne d'attaque de la Phase A
   depuis Kali contre l'architecture Zero Trust.
7. **Validation finale** *(à venir)* — Audit des règles OPNsense,
   checklist de reproductibilité, tableau avant/après.

## Résultats (Phase A — baseline)

| Attaque                        | Statut   | Vulnérabilité exploitée                     |
|---------------------------------|----------|-----------------------------------------------|
| Scan réseau                     | Réussi   | Absence de filtrage, tous les services visibles |
| Force brute SSH                 | Réussi (2 min) | Mot de passe faible et prévisible        |
| Extraction de configuration     | Réussi   | Identifiants stockés en clair                |
| Accès direct base de données    | Réussi   | Base de données non filtrée réseau           |
| Mouvement latéral               | Réussi   | Absence de segmentation                      |
| Interception de trafic          | Réussi   | Absence de chiffrement TLS                   |

Le tableau « après » sera complété à l'issue du Step 12 (voir
[docs/tests.md](zero-trust-lab/docs/tests.md)).

## Documentation

- [Architecture](zero-trust-lab/docs/architecture.md)
- [Tests de validation](zero-trust-lab/docs/tests.md)
- [Déploiement Wazuh](zero-trust-lab/docs/wazuh.md)
- [Limites connues](zero-trust-lab/docs/limits.md)
- [Guide de publication](zero-trust-lab/GIT-SETUP.md)

## Stack technique

OPNsense · VirtualBox (réseaux internes isolés) · Debian 13 · MariaDB ·
Nextcloud · Keycloak 26 · Traefik 3.3 · Authelia 4.38 · Wazuh 4.14.9 ·
Kali Linux · Nmap · Hydra · CUPP · Wireshark

## Avertissement sur les identifiants

Ce lab tourne entièrement dans des réseaux privés non routables
(`10.10.x.x`, `192.168.x.x`). Aucun secret d'infrastructure n'est publié
(clés, hashs, mots de passe admin : placeholders — voir `.env.example`) ; les
mots de passe volontairement faibles des comptes de test sont conservés car
ils sont au cœur de la démonstration de la force brute.

## Auteure

Imane Arrach
