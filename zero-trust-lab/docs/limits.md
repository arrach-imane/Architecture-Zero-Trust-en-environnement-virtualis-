# Limites connues

- Keycloak en `start-dev` avec base H2 (pas de production : pas de PostgreSQL, pas de HA).
- Vérification TLS assouplie côté Nextcloud (certificat auto-signé).
- Une seule VM IAM : point unique de défaillance.
- Flux de vulnérabilités Wazuh hors ligne.
- La qualité du TOTP dépend de l'application d'authentification utilisée.
- Les VMs peuvent se bloquer (soft lockup) quand la RAM de l'hôte est saturée.
- Comptes de test volontairement faibles (démo de la force brute) ; aucun secret d'infrastructure n'est publié.
- L'audit des règles OPNsense (Step 13) a montré que des règles « destination étoile » trop larges peuvent apparaître sur n'importe quelle interface, pas seulement celle visiblement liée à un incident : un audit ponctuel ne remplace pas une revue périodique des règles.
