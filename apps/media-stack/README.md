# Media Stack

Stack complète pour la gestion de médias automatisée.

## Services

| Service | Port | URL | Description |
|---------|------|-----|-------------|
| qBittorrent | 8080 | torrent.cooper-host.com | Client torrent |
| Prowlarr | 9696 | prowlarr.cooper-host.com | Gestionnaire d'indexeurs |
| Radarr | 7878 | radarr.cooper-host.com | Gestion des films |
| Sonarr | 8989 | sonarr.cooper-host.com | Gestion des séries |
| Autobrr | 7474 | autobrr.cooper-host.com | Auto-téléchargement IRC |
| qBit Manage | 8181 | qbitmanage.cooper-host.com | Gestion avancée de qBittorrent |

## Prérequis

1. Réseau Docker `proxy` créé (pour Traefik) :
   ```bash
   docker network create proxy
   ```

2. Volume de stockage `/storage/plex/Media` accessible

3. Certificats TLS configurés dans Traefik (letsencrypt)

## Déploiement

```bash
cd apps/media-stack
docker compose up -d
```

## Configuration initiale

### 1. Prowlarr
- Ajouter les indexeurs (trackers)
- Configurer les connecteurs vers Radarr/Sonarr

### 2. qBittorrent
- Accéder à l'interface web
- Mot de passe par défaut dans les logs : `docker logs qbittorrent`
- Configurer les chemins de téléchargement

### 3. Radarr/Sonarr
- Configurer les profils de qualité
- Ajouter les connecteurs vers qBittorrent (via Prowlarr)
- Définir les chemins : `/data/movies` pour Radarr, `/data/tv` pour Sonarr

### 4. Autobrr
- Configurer les IRC channels pour les trackers
- Créer les filtres de téléchargement

### 5. qBit Manage
- Éditer `config/qbitmanage/config.yml`
- Configurer les règles de gestion des torrents

## Volumes

| Conteneur | Chemin | Description |
|-----------|--------|-------------|
| Tous | `./config/<service>` | Configuration persistante |
| Tous | `/storage/plex/Media:/data` | Données media partagées |

## Problèmes potentiels

1. **Réseau proxy manquant** : Créer avec `docker network create proxy`
2. **Permissions** : Vérifier que PUID/PGID correspondent aux droits sur `/storage/plex/Media`
3. **Ports 6881** : Doivent être ouverts sur le firewall pour les torrents
4. **Espace disque** : Prévoir assez d'espace pour les téléchargements
5. **Premier lancement** : Les services peuvent prendre 1-2 min à démarrer (healthchecks)
