# Zero Trust Lab

Architecture Zero Trust complète bâtie dans un environnement virtualisé
(VirtualBox) : segmentation réseau, gestion centralisée des identités,
reverse proxy avec authentification multifacteur et supervision —
démontrée par une chaîne d'attaque réelle rejouée avant et après
sécurisation.

## Contexte

Projet réalisé dans le cadre du module Cloud Computing et Sécurité,
FST Marrakech, filière SIT.

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

| Segment     | Sous-réseau    | Rôle                                         |
|-------------|----------------|-----------------------------------------------|
| MGMT        | 10.10.1.0/24   | Administration du lab                        |
| IAM / DMZ   | 10.10.2.0/24   | Identité (Keycloak, Authelia, Traefik)       |
| APPS        | 10.10.3.0/24   | Applications (Nextcloud)                     |
| DB          | 10.10.4.0/24   | Base de données (MariaDB)                    |
| ATTACKER    | 10.10.5.0/24   | Poste d'attaque (Kali)                       |
| MONITORING  | 10.10.6.0/24   | Supervision (Wazuh)                          |

Chaque segment ne communique qu'avec les segments strictement nécessaires
à sa fonction (ex. le segment DB n'a aucune règle sortante : il ne fait que
répondre aux requêtes déjà autorisées en entrée depuis APPS).

## Démarche

1. **Step 1-4** — Phase A : mise en place du réseau plat, de Server-DB et
   Server-Nextcloud, et chaîne d'attaque de référence (scan Nmap → OSINT/
   CUPP → brute force SSH Hydra → extraction des identifiants → accès
   direct à la base de données → interception de trafic HTTP en clair).
2. **Step 5-6** — Phase B : segmentation réseau en six segments OPNsense et
   définition des règles de pare-feu selon le principe de moindre
   privilège, avec vérification positive/négative par segment.
3. **Step 7-9** *(en cours)* — Déploiement de la brique IAM : Keycloak pour
   la gestion des identités, Traefik en reverse proxy, Authelia pour
   l'authentification multifacteur devant les applications.
4. **Step 10** *(à venir)* — Supervision avec Wazuh sur le segment
   MONITORING, flux de remontée strictement unidirectionnel.
5. **Step 11-14** *(à venir)* — Durcissement final, rejeu complet de la
   chaîne d'attaque de la Phase A sur l'architecture Zero Trust, tableau
   de comparaison avant/après, conclusion.

## Résultats (Phase A — baseline)

| Attaque                        | Statut   | Vulnérabilité exploitée                     |
|---------------------------------|----------|-----------------------------------------------|
| Scan réseau                     | Réussi   | Absence de filtrage, tous les services visibles |
| Force brute SSH                 | Réussi (2 min) | Mot de passe faible et prévisible        |
| Extraction de configuration     | Réussi   | Identifiants stockés en clair                |
| Accès direct base de données    | Réussi   | Base de données non filtrée réseau           |
| Mouvement latéral               | Réussi   | Absence de segmentation                      |
| Interception de trafic          | Réussi   | Absence de chiffrement TLS                   |

Le tableau « après » correspondant sera complété à l'issue du Step 12, une
fois la chaîne d'attaque rejouée contre l'architecture Zero Trust.

## Documentation complète

Voir [docs/](docs/) pour les chapitres détaillés du rapport (contexte,
commandes exactes, incidents rencontrés et leur résolution, captures
d'écran référencées).

## Stack technique

OPNsense · VirtualBox (réseaux internes isolés) · Debian 13 · MariaDB ·
Nextcloud · Keycloak · Traefik · Authelia · Wazuh *(à venir)* · Kali Linux ·
Nmap · Hydra · CUPP · Wireshark

## Avertissement sur les identifiants

Ce lab tourne entièrement dans des réseaux privés non routables
(`10.10.x.x`, `192.168.x.x`). Les mots de passe d'infrastructure réels ont
été retirés des fichiers publiés ici (voir [docs/README.md](docs/README.md)
pour le détail) ; les mots de passe volontairement faibles attribués aux
comptes de test, eux, sont conservés tels quels car ils sont au cœur de la
démonstration de l'attaque par force brute.

## Auteure

Imane Arrach
