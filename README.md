# HAProxy et Keepalived pour Kubernetes

Dans le projet d’équipe SLLYM, HAProxy servait à répartir le trafic vers Kubernetes. La même base était déployée avec deux configurations, l’une pour les accès internes et l’autre pour le point d’entrée en DMZ. Deux pipelines GitLab permettaient de déployer chaque configuration séparément.

## Accès internes et bascule

- Le [fragment Kubernetes](ansible/roles/haproxy/templates/conf.d/10-kubernetes.cfg.j2) sépare le trafic de l’API et celui de l’ingress, qui reçoit les requêtes vers les applications.
- La [configuration Keepalived](ansible/roles/keepalived/templates/keepalived.conf.j2) et la [sonde HAProxy](ansible/roles/keepalived/templates/check_haproxy.sh.j2) font basculer l’adresse virtuelle si le répartiteur actif échoue à ses contrôles.

## Point d’entrée DMZ

- Le [fragment Keycloak](ansible/roles/haproxy/templates/conf.d/10-keycloak-public.cfg.j2) illustre le filtrage des accès externes et la vérification du certificat de l’ingress.
- La [configuration HAProxy de l’agent Coraza](ansible/roles/haproxy/templates/conf.d/05-coraza-spoa.cfg.j2) et les [échanges avec le pare-feu applicatif](ansible/roles/coraza_spoa/templates/coraza.cfg.j2) montrent l’inspection des en-têtes et des paramètres transmis par HAProxy.

## Vérification avant déploiement

Le [rendu de validation](ansible/extraits/validate-render.yml) assemble la configuration et la soumet à `haproxy -c` pour détecter les erreurs avant un déploiement.

Ces extraits du PFE d’équipe illustrent les deux configurations et leur méthode de validation. Pour exécuter la validation, il faut aussi le modèle HAProxy de base, les variables Ansible, l’inventaire et les certificats de l’environnement cible.
