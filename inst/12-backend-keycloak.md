########################################################################################################
## KEYCLOAK
## 

- Set up the docker container scripts

```{bash}
mkdir --parents $DIGITAGGER_HOME/inst/keycloak
cat << EOF | sudo tee $DIGITAGGER_HOME/inst/keycloak/docker-compose.yml
version: "3"
services:
  postgres:
    image: postgres:14.1
    container_name: clariah-pg   
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: ${POSTGRESQL_KEYCLOAK_USER}
      POSTGRES_PASSWORD: ${POSTGRESQL_KEYCLOAK_PWD}
    ports:
      - 5433:5432
    volumes:
      - ${POSTGRESQL_KEYCLOAK_PGDATA}/data/postgres:/var/lib/postgresql/data 
    restart: unless-stopped
  keycloak:
    image: jboss/keycloak:16.1.0
    container_name: clariah-keycloak  
    depends_on:
      - postgres    
    environment:
      DB_VENDOR: POSTGRES
      DB_ADDR: postgres
      DB_DATABASE: keycloak
      DB_USER: ${POSTGRESQL_KEYCLOAK_USER}
      DB_SCHEMA: public
      DB_PASSWORD: ${POSTGRESQL_KEYCLOAK_PWD}
      KEYCLOAK_LOGLEVEL: INFO
      ROOT_LOGLEVEL: INFO
      PROXY_ADDRESS_FORWARDING: 'true'
      JDBC_PARAMS: 'useSSL=false'
    ports:
      - 28080:8080
    volumes:
      - ${DIGITAGGER_HOME}/inst/keycloak/adminlte-keycloak-theme/adminlte:/opt/jboss/keycloak/themes/adminlte
    restart: unless-stopped
EOF
```

- Start up the container

```{bash}
sudo docker-compose -f $DIGITAGGER_HOME/inst/keycloak/docker-compose.yml up --detach 
```

- Add keycloak admin user

```{bash}
sudo docker exec -it --env-file=$DIGITAGGER_HOME/inst/.env clariah-keycloak bash /opt/jboss/keycloak/bin/add-user-keycloak.sh -u $KEYCLOAK_ADMIN_USER -p $KEYCLOAK_ADMIN_PWD 
## /opt/jboss/keycloak/bin
sudo docker restart clariah-keycloak
```

- In case you want to inspect the logs or pull it down

```{bash}
sudo docker ps --all
sudo docker-compose -f $DIGITAGGER_HOME/inst/keycloak/docker-compose.yml logs
## Pull it down if needed for debugging
sudo docker-compose -f $DIGITAGGER_HOME/inst/keycloak/docker-compose.yml down
sudo docker rm clariah-pg   
sudo docker rm clariah-keycloak 
## Inspect logs
sudo docker logs clariah-keycloak 
sudo sh -c "truncate -s 0 /var/lib/docker/containers/*/*-json.log"
cd /var/lib/docker/containers/
```

- Configuration
    - <img src="https://avatars.githubusercontent.com/u/62653831?s=400&u=5a22128cd3bfae334b8a2c34d59261ab4bdb7cac&v=4" alt="logo" style="width:90px;height:90px"><div class="kc-logo-text"><span>digitagger admin</span></div>
    - <img src="https://avatars.githubusercontent.com/u/62653831?s=400&u=5a22128cd3bfae334b8a2c34d59261ab4bdb7cac&v=4" alt="logo" style="width:90px;height:90px"><div class="kc-logo-text"><span>digitagger apps</span></div>    
    - clients: 
        - rstudio-client confidential with `Valid Redirect URIs`: https://rstudio.digitagger.org/*
        - shiny-client: confidential with `Valid Redirect URIs`: https://apps.digitagger.org/app/*
        - inception-client: confidential with `Valid Redirect URIs`: https://inception.digitagger.org/*
        - cantaloupe-client confidential with `Valid Redirect URIs`: https://iiif.digitagger.org/administration/*

- Dev on startup:

- Following scripts should be put at /opt/jboss/startup-scripts/

```
sudo docker exec -it --env-file=$DIGITAGGER_HOME/inst/.env clariah-keycloak bash
KCADM="/opt/jboss/keycloak/bin/kcadm.sh"
${KCADM} config credentials --server http://localhost:8080/auth --realm master --user ${KEYCLOAK_ADMIN_USER} --password ${KEYCLOAK_ADMIN_PWD}
${KCADM} create clients -r dev -s clientId=test -s enabled=true -s publicClient=false -s 'redirectUris=["https://apps.digitagger.org/app/*"]' -s clientAuthenticatorType=client-secret -s secret=${KEYCLOAK_CLIENT_SHINY_SECRET}
${KCADM} get clients -r dev --fields id,clientId

```


