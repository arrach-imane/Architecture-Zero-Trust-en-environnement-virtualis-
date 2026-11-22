# Limites connues

- Keycloak en `start-dev` avec base H2 (pas de production : pas de PostgreSQL, pas de HA).
- Vérification TLS assouplie côté Nextcloud (certificat auto-signé).
- Une seule VM IAM : point unique de défaillance.
- Flux de vulnérabilités Wazuh hors ligne.
- La qualité du TOTP dépend de l'application d'authentification utilisée.
- Les VMs peuvent se bloquer (soft lockup) quand la RAM de l'hôte est saturée.
- Comptes de test volontairement faibles (démo de la force brute) ; aucun secret d'infrastructure n'est publié.
