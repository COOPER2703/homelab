# Homelab IaC

Infrastructure as Code pour le serveur homelab. Gère le provisioning (Ansible) et le déploiement des services Docker (docker-compose + Traefik).

## Language

**App**:
Un service Docker déployable, défini par un `docker-compose.yml` dans son propre dossier sous `apps/`.
_Avoid_: service, container, stack

**Stack**:
Un groupe d'apps liées partageant un même `docker-compose.yml`.
_Avoid_: compose, project

**Proxy**:
Réseau Docker partagé (`proxy`) reliant Traefik à toutes les apps exposées. Traefik route le trafic via les labels Docker.
_Avoid_: traefik-net, frontend network

**IaC**:
Le repo entier contenant les playbooks Ansible et les apps Docker.
_Avoid_: infra, config repo

**Volume**:
Espace de stockage persistant monté dans un container. Chemin absolu sur l'hôte (`/opt/apps/` pour les configs, `/storage/` pour les données lourdes).
_Avoid_: mount, bind mount
