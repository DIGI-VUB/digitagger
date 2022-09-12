
########################################################################################################
## PostgreSQL backend
##   - For Cantaloupe
##   - For DB storage of apps
## 

## PostgreSQL

```{bash}
sudo apt update
sudo apt install -y postgresql postgresql-contrib postgresql-client-common libpq5
sudo service postgresql status
```

## Cantaloupe iiif backend

```{bash}
sudo mkdir /mnt/digidb/postgres
sudo chown postgres:postgres /mnt/digidb/postgres
sudo su postgres
createuser --echo --superuser digi 
psql -c "ALTER USER digi PASSWORD '$POSTGRESQL_DIGI_PWD';"
psql -c "CREATE TABLESPACE digidb OWNER digi LOCATION '/mnt/digidb/postgres'"
createdb iiif --encoding=utf-8 --owner=digi --tablespace=digidb
#psql --dbname=digidb -c "GRANT CONNECT ON DATABASE iiif TO digidb;"
#psql --dbname=digidb -c "GRANT USAGE ON SCHEMA public TO digidb;"
#psql --dbname=digidb -c "GRANT USAGE ON SCHEMA keycloak TO digidb;"
#psql --dbname=digidb -c "GRANT SELECT ON ALL TABLES IN SCHEMA public TO digidb;"
#psql --dbname=digidb -c "ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO digidb;"
exit
```

## Digi DB for o.a. Getuigenissen geocoding

- Install PostGIS

```{bash}
sudo apt install postgis postgresql-12-postgis-3
```

- Allow access to PostgreSQL DB
    - Set listen_addresses = '*' in /etc/postgresql/12/main/postgresql.conf
    - Add in /etc/postgresql/12/main/pg_hba.conf
          host    all             all              0.0.0.0/0              md5
          host    all             all              ::/0                   md5

```{bash}
sudo service postgresql restart
```

- Make user digivub which owns database digi which was dumped from a local DB
- Make user qgis which has read rights on some getuigenissen tables

```{bash}
sudo su postgres
createuser --echo digivub
psql -c "ALTER USER digivub PASSWORD '$POSTGRESQL_DIGIVUB_PWD';"
createuser --echo qgis
psql -c "ALTER USER qgis PASSWORD '$POSTGRESQL_QGIS_PWD';"
createdb digi --encoding=utf-8 --owner=digivub --tablespace=digidb
## upload dump from own PostgreSQL (pg_dump --username=digivub digi > pg_digi.sql) on sharepoint and import at this server
psql --file=/home/ubuntu/pg_digi.sql --dbname=digi postgres
```

```{bash}
sudo su postgres
psql -c 'SHOW config_file'
exit
sudo vim /etc/postgresql/12/main/pg_hba.conf
host all all 10.10.29.0/24 trust
# Remote access
host    all             all              0.0.0.0/0                       md5
host    all             all              ::/0                            md5

sudo systemctl restart postgresql
```


## PyBOSSA

```{bash}
sudo apt update
sudo apt install -y postgresql postgresql-server-dev-all libpq-dev python3-psycopg2 libsasl2-dev libldap2-dev libssl-dev python3-dev
sudo apt install -y python-dev build-essential libjpeg-dev libssl-dev libffi-dev
sudo apt install -y dbus libdbus-1-dev libdbus-glib-1-dev libldap2-dev libsasl2-dev
sudo apt install -y redis-server
sudo apt install -y libxml2-dev libxslt1-dev python-dev

cd $HOME
pwd
git clone --recursive https://github.com/Scifabric/pybossa
cd pybossa
git checkout v4.0.1
pip install pip
pip install -r requirements.txt
```
