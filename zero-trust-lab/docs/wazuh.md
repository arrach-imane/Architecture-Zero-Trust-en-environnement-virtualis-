# Déploiement Wazuh

- Version 4.14.9, single-node Docker, Debian 13 sans bureau, 6 Go de RAM.
- Prérequis noyau : `vm.max_map_count=262144`.
- Certificats générés avec l'outil officiel ; **mots de passe par défaut modifiés** (indexer, API, dashboard) — valeurs non publiées.
- Agents : server-iam (001), server-nextcloud (002), server-db (003), kali-attacker (004).

## Flux réseau

Les agents ouvrent des connexions sortantes vers 10.10.6.10 (TCP 1514 événements, 1515 enrôlement). Aucune règle sur OPT5 : par défaut, tout est refusé depuis MONITORING.

## Limites

Le flux de vulnérabilités ne peut pas se mettre à jour (pas d'accès sortant). L'intégration syslog OPNsense n'a volontairement pas été faite (optionnelle).
