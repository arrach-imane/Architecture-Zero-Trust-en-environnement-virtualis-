# Documentation

- `step3-step4-rapport.tex` — Phase A : réseau plat, mise en place de
  Server-DB et Server-Nextcloud, chaîne d'attaque complète (reconnaissance
  Nmap → OSINT/CUPP → brute force SSH Hydra → exfiltration MariaDB →
  interception de trafic HTTP).
- `step5-step6-rapport.tex` — Phase B, étapes 5-6 : segmentation réseau en
  six segments OPNsense, conception et implémentation des règles de
  pare-feu selon le principe de moindre privilège.

> **Note sur les identifiants** : les mots de passe réels d'infrastructure
> (base de données, compte administrateur) ont été remplacés par des
> placeholders (`DB_PASSWORD_REDACTED`, `ADMIN_PASSWORD_REDACTED`) avant
> publication. Les mots de passe *faibles* attribués aux comptes de test
> (`Jdupont123`, `Alaoui12345678`, etc.) sont volontairement conservés tels
> quels : ils sont au cœur de la démonstration pédagogique de l'attaque par
> force brute et ne correspondent à aucun compte réel réutilisé ailleurs.

