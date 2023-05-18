# KTHB Dialog
KTH Bibliotekets tjänst för att utföra dialog med användare(t ex röstning, matchmaking etc)

## Funktioner
Startas i en Dockercontainer

###
Deploy via github actions som anropar en webhook

#### Dependencies

Node 16.13.2

##### Installation

1.  Skapa folder på server med namnet på repot: "/local/docker/dialogtools"
2.  Skapa och anpassa docker-compose.yml i foldern
```
version: '3.6'

services:
  dialogtools:
    container_name: dialogtools
    image: ghcr.io/kth-biblioteket/dialogtools:${REPO_TYPE}
    restart: always
    depends_on:
      - dialogtools-db
    env_file:
      - dialogtools.env
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dialogtools.rule=Host(`${DOMAIN_NAME}`) && PathPrefix(`${PATHPREFIX}`)"
      - "traefik.http.routers.dialogtools.entrypoints=websecure"
      - "traefik.http.routers.dialogtools.tls=true"
      - "traefik.http.routers.dialogtools.tls.certresolver=myresolver"
    networks:
      - "apps-net"

  dialogtools-db:
    image: mysql:8.0
    volumes:
      - persistent-dialogtools-db:/var/lib/mysql
      - ./db:/docker-entrypoint-initdb.d
    restart: unless-stopped
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    networks:
      - "apps-net"

volumes:
  persistent-dialogtools-db:

networks:
  apps-net:
    external: true
```
3.  Skapa och anpassa .env(för composefilen) i foldern
```
PATHPREFIX=/dialogtools
DOMAIN_NAME=apps-ref.lib.kth.se
REPO_TYPE=ref
API_ROUTES_PATH=/api/v1
DB_DATABASE=dialogtools
DB_USER=dialogtools
DB_PASSWORD=XXXXXX
DB_ROOT_PASSWORD=XXXXXX
```
4.  Skapa och anpassa dialogtools.env (för applikationen) i foldern
```

PORT=80
APP_PATH=/dialog
API_PATH=/api/v1
APIKEY=xxxxxx
LDAPAPIPATH=ldap-api/api/v1
SMTP_HOST=relayhost.sys.kth.se
SMTP_HOST_NEW=relayhost.sys.kth.se
MAILFROM_NAME_SV=KTH Bibliotekets löftesinsamling
MAILFROM_NAME_EN=KTH Library promise
MAILFROM_ADDRESS=noreply@apps-ref.lib.kth.se
MAILFROM_SUBJECT_SV=Ditt lämnade löfte.
MAILFROM_SUBJECT_EN=Your promise.
EDGE_MAIL_ADDRESS=tholind@kth.se
SECRET=xxxxxx
LDAPAPIKEYREAD=xxxxxx
DATABASEHOST=ref.lib.kth.se
DATABASE=dialogtools
DATABASEUSER=dialogtools
DATABASEPWD=xxxxxx
DATABASEROOTPWD=xxxxxx
GITHUBTOKEN=xxxxxx
LOG_LEVEL=debug
NODE_ENV=development
IMAGE_FORMAT=jpg
AUTHORIZEDGROUPS=pa.anstallda.T.TR;pa.anstallda.M.MOE
LABELTYPE=fa-heart
```
5.  Skapa folder "local/docker/dialogtools/imagesbank"
6.  Skapa folder "local/docker/dialogtools/dbinit"
7. Skapa init.sql från repots dbinit/init.sql
8. Skapa deploy_ref.yml i github actions
9. Skapa deploy_prod.yml i github actions
10. Github Actions bygger en dockerimage i github packages
11. Starta applikationen med docker compose up -d --build i "local/docker/dialogtools"

