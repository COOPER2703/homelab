# Obsidian LiveSync (CouchDB)

Hub de synchronisation du coffre Obsidian, exposé sur `https://obsidian.cooper-host.com`.

## Identifiants de connexion

| Champ | Valeur |
|---|---|
| URI | `https://obsidian.cooper-host.com` |
| Base de données | `obsidiannotes` |
| Utilisateur | `obsidian` (ou `vault_obsidian_admin_user`) |
| Mot de passe | `vault_obsidian_admin_password`, dans l'Ansible Vault |

Le mot de passe n'est jamais versionné : il vit dans `ansible/group_vars/servers.yml` (chiffré) et est rendu dans `/opt/apps/obsidian/.env` en `0600`.

## Configuration côté client

1. Installer le plugin communautaire **Self-hosted LiveSync** dans Obsidian.
2. Renseigner l'URI, l'utilisateur, le mot de passe et le nom de base ci-dessus, puis tester la connexion.
3. Choisir une **passphrase de chiffrement de bout en bout**. Elle est identique sur tous les appareils, n'est jamais stockée sur le serveur et n'existe pas dans ce repo.
4. Au premier appareil, créer la base `obsidiannotes` depuis le plugin ; les appareils suivants s'y connectent.

La base n'est pas créée par Ansible : le plugin s'en charge, ce qui évite de dépendre de l'URL publique pendant le premier déploiement.

## Déploiement

```bash
cd ansible
ansible-playbook main.yml --ask-vault-pass
```

Les répertoires `/opt/apps/obsidian/{data,etc}` sont créés en `1000:1000`, l'uid sous lequel tourne le conteneur. `etc` est monté sur `/opt/couchdb/etc/local.d` : c'est là que CouchDB écrit `docker.ini` (compte admin) et que sont lues les options de `10-livesync.ini`.

Aucun port n'est publié : seul Traefik joint le conteneur sur le réseau `proxy`. Le DNS est déjà couvert par le wildcard `*.cooper-host.com` (proxy Cloudflare), il n'y a pas d'enregistrement à créer.

## Pièges connus

- `COUCHDB_USER` / `COUCHDB_PASSWORD` ne sont appliqués **qu'au premier démarrage**, quand `/opt/apps/obsidian/etc/docker.ini` n'existe pas encore. Modifier `.env` ensuite ne change rien. La rotation passe par l'API, puis par la mise à jour de `vault_obsidian_admin_password` avec la même valeur (sinon le prochain déploiement réécrit l'ancien mot de passe et le healthcheck échoue) :

  ```bash
  curl -u obsidian:ancien_mot_de_passe -X PUT \
    'https://obsidian.cooper-host.com/_node/_local/_config/admins/obsidian' \
    -H 'Content-Type: application/json' \
    -d '"nouveau_mot_de_passe"'
  ```

- **Pas de sauvegarde serveur.** Chaque appareil Obsidian détient une réplique du coffre ; perdre le hub coûte une reconfiguration, pas les notes.
- **Cloudflare** : en cas d'erreur 524 pendant une grosse synchronisation, activer l'option « Use Request API » du plugin.
- `https://obsidian.cooper-host.com/_utils/` donne accès à l'interface d'administration CouchDB. Elle exige le compte admin et n'est pas protégée par une couche supplémentaire.
- Le lockout CouchDB est en `mode = warn` : derrière Traefik, toutes les requêtes ont la même IP source, un lockout `enforce` bloquerait tout le monde.
- Les origines CORS autorisées couvrent le client desktop (`app://obsidian.md`) et mobile (`capacitor://localhost`, `http://localhost`). Ajouter une origine dans `10-livesync.ini` si un nouveau type de client apparaît : le redéploiement redémarre le conteneur quand ce fichier change.
