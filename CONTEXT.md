# Homelab IaC

Infrastructure as Code du serveur homelab : provisioning Ansible et déploiement de services Docker via des templates de compose, exposés par Traefik.

## Language

**App**:
Un service Docker déployable, décrit par un dossier `ansible/templates/<app>/` contenant un `docker-compose.yml.j2` et un `deploy.yml`.
_Avoid_: service, container, stack

**Stack**:
Un groupe d'apps liées qui partagent un même `docker-compose.yml`.
_Avoid_: compose, project

**Proxy**:
Le réseau Docker externe `proxy` qui relie Traefik aux apps exposées ; Traefik route le trafic à partir des labels Docker.
_Avoid_: traefik-net, frontend network

**IaC**:
Le contenu de ce repo : playbooks Ansible, templates d'apps et documentation.
_Avoid_: infra, config repo

**Volume**:
Un espace de stockage persistant monté dans un conteneur. Chemin absolu sur l'hôte : `/opt/apps/<app>/` pour les configurations, `/mnt/storage/` pour les données lourdes.
_Avoid_: mount, bind mount

**Coffre**:
Le répertoire de notes d'un utilisateur Obsidian, répliqué entre ses appareils.
_Avoid_: vault (réservé à Ansible Vault)

**Ansible Vault**:
Le fichier chiffré qui contient les secrets d'exploitation, notamment `ansible/group_vars/servers.yml`.
_Avoid_: coffre, secrets file

**LiveSync**:
Le plugin Obsidian qui réplique un coffre vers une base CouchDB.
_Avoid_: sync, obsidian sync

**Hub LiveSync**:
L'instance CouchDB exposée sur `obsidian.cooper-host.com`, point de rendez-vous des appareils d'un coffre.
_Avoid_: serveur obsidian, base obsidian
