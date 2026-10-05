## credits
[spad](https://www.spad.uk/posts/practical-configuration-of-traefik-as-a-reverse-proxy-for-docker-updated-for-2023/)

## Pre requis
  
   - livebox Orange 
   se connecter 192.168.1.1 admin Se$ame!1
   dans l'onglet Réseau
   - DHCP attribuer une adresse fixe  au serveur : 192.168.1.102
   - donner un nom au serveur : asus
   - rechercher l'adresse IP de la box , dans le naviguateur whatismyip.com  -> 90.92.38.140
      cette adresse peut changer !
   -  un nom de domaine cbillon.ovh  géré par OVH
   se connecter ovh.com bc1678707-ovh qoznof-xesu6-kutZuc

    - mettre à jour les zones DNS cbillon.ovh et *.cbillon.ovh ave l'adresse de la box : 90.92.38.140  

  si l'adresse IP de la box a changé il faut faire ma lamise jour
  la prodédure dyndns-OVH le fait automatuqment (non testé)

## Configuration

- traefik itself
  - compose.yml
  - .env
  - traefik.yml
  - configs
    - nas.yml

- container

## .env

it requires to escape certain characters such as $

## create file acme.json

If you try and mount a file that doesn’t exist on the host into a container, Docker will instead create a folder on the host with the same name, and then likely throw an error and fail to create the container. If this happens, delete the folder, create the file, then recreate the container.

```
touch acme.json
sudo chown root:root acme.json
sudo chmod 600 acme.json

```
## traefik.yml

### providers

- exposeByDefault: false; must declare on each container ; set true to make all containers available by default, but it’s probably not a good idea. 
- network: proxy  use the proxy network by default to reach backend containers 
- <servicename>.cbillon.ovh set up a defaultRule for container hostnames; if you don’t specify a hostname for a container, it will default to <servicename>.cbillon.ovh
- file : provider for non docker backend (nas)
  set watch = true dynamically update otherwise it requires a restart 

### log

Access logging is optional but frequently useful, here we also tell it to make sure it keeps the User-Agent field and you can see a complete list of available fields to keep, drop, or redact [link](https://doc.traefik.io/traefik/observability/access-logs/#limiting-the-fieldsincluding-headers).

log
- filepath : if missing stdout by default
- level: set to INFO DEBUG WARNING ERROR

## Creation external network proxy

```
  docker network create proxy

```
## pour tester
lancer traefik docker compose up -d 
puis dans le naviguateur
- whoami.cbillon.ovh affiche l'adresse ip du conteneur
- traefik.cbillon.ovh/dashboard/ (ne pas oublier le /)  authentification cb/sesame
  la configuration identifiant/mot de passe se trouve dans dans la configuration .env

## moodle

pour integrer Moodle
dans le container docker_moodle-web ajouter

```
  networks:
      - proxy
      - docker_moodle
    labels:
      - traefik.enable=true
      - traefik.http.routers.moodle.rule=Host(`moodle.cbillon.ovh`)
      - traefik.http.routers.moodle.entrypoints=https
      - traefik.http.services.moodle.loadbalancer.server.port=8088
      - traefik.http.routers.moodle.tls.certresolver=letsencrypt
      - traefik.http.routers.moodle.tls.domains[0].main=cbillon.ovh
      - traefik.http.routers.moodle.tls.domains[0].sans=*.cbillon.ovh
      
```