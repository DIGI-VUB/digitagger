
########################################################################################################
## Inception
## 

## Java

```{bash}
sudo apt update
sudo apt install -y openjdk-17-jdk openjdk-17-jre
```

## MariaDB

```{bash}
sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
sudo add-apt-repository 'deb [arch=amd64] http://mariadb.mirror.globo.tech/repo/10.5/ubuntu focal main'
sudo apt update
sudo apt install -y mariadb-server mariadb-client
```

- Change location of mariadb to the backed up volume

```{bash}
mkdir --parents $DIGITAGGER_HOME/inst/mariadb
mkdir --parents $MARIADB_DATADIR
cat /etc/mysql/mariadb.conf.d/50-server.cnf 
sudo systemctl status mariadb
sudo systemctl stop mariadb
sudo rsync -av /var/lib/mysql $MARIADB_DATADIR
sudo chown -R mysql:mysql $MARIADB_DATADIR

cat << EOF | sudo tee /etc/my.cnf
[server]

[mysqld]
user                    = mysql
pid-file                = /run/mysqld/mysqld.pid
basedir                 = /usr
datadir                 = ${MARIADB_DATADIR}
tmpdir                  = /tmp
bind-address            = 127.0.0.1

#key_buffer_size        = 128M
#max_allowed_packet     = 1G
#thread_stack           = 192K
#thread_cache_size      = 8
#myisam_recover_options = BACKUP
#max_connections        = 100
#table_cache            = 64
expire_logs_days        = 10

# MySQL/MariaDB default is Latin1, but in Debian we rather default to the full
# utf8 4-byte character set. See also client.cnf
character-set-server  = utf8mb4
collation-server      = utf8mb4_general_ci
character-set-client-handshake = FALSE
innodb_large_prefix=true
innodb_file_format=barracuda
innodb_file_per_table=1
innodb_strict_mode=1
innodb_default_row_format='dynamic'

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4
EOF

sudo systemctl start mariadb
sudo systemctl status mariadbq
```

## Create Inception DB

- Create DB inception with user inception as owner, check if MariaDB server is configured for 4 Byte UTF-8 (utf8mb4, utf8mb4_bin)

```{bash}
sudo mysql -u root -e "SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';"
cat << EOF | tee $DIGITAGGER_HOME/inst/mariadb/create-db.sh
DROP DATABASE IF EXISTS inception;
DROP USER IF EXISTS 'inception'@localhost;
CREATE DATABASE inception DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
SHOW DATABASES;
CREATE USER 'inception'@localhost IDENTIFIED BY '$MARIADB_INCEPTION_PWD';
GRANT ALL PRIVILEGES ON inception.* TO 'inception'@localhost;
FLUSH PRIVILEGES;
EOF
sudo mysql -u root < $DIGITAGGER_HOME/inst/mariadb/create-db.sh
rm $DIGITAGGER_HOME/inst/mariadb/create-db.sh
```

- Test connection `mysql -u inception --port=3306 --host=localhost -p `
- In case you need backup / restore

``` 
sudo mysqldump --all-databases -u root > $DIGITAGGER_HOME/inst/mariadb/db.sql
sudo mysql -u root < $DIGITAGGER_HOME/inst/mariadb/db.sql
```

## Inception configuration to be run as a service

```{bash}
sudo mkdir /srv/inception
sudo cat /srv/inception/settings.properties
cat << EOF | sudo tee /srv/inception/settings.properties
database.url=jdbc:mariadb://localhost:3306/inception?useSSL=false&serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8
database.username=inception
database.password=$MARIADB_INCEPTION_PWD
# 60 * 60 * 24 * 30 = 30 days
backup.keep.time=2592000
# 60 * 5 = 5 minutes
backup.interval=300
backup.keep.number=10
useSSL=false
allowPublicKeyRetrieval=true
serverTimezone=UTC
server.port=18080
server.use-forward-headers=true
server.tomcat.internal-proxies=127\.0\.[0-1]\.1
server.tomcat.remote-ip-header=x-forwarded-for
server.tomcat.accesslog.request-attributes-enabled=true
server.tomcat.protocol-header=x-forwarded-proto
server.tomcat.protocol-header-https-value=https
server.use-forward-headers=true
wicket.core.csrf.enabled=false
wicket.core.csrf.no-origin-action=allow
remote-api.enabled=true
annotation.default-preferences.page-size=2000
# Authenticate using keycloak instead of using the Inception login
auth.mode = preauth
auth.preauth.header.principal = remote_user
auth.preauth.newuser.roles = ROLE_PROJECT_CREATOR
auth.preauth.logoutUrl = https://iam.digitagger.org/auth/realms/dev/protocol/openid-connect/logout
auth.user.admin.roles = ROLE_ADMIN
sharing.invites.enabled = true
sharing.invites.invite-base-url = https://inception.digitagger.org
EOF
sudo chown -R www-data:www-data /srv/inception
```

## Inception service

```
sudo wget https://github.com/inception-project/inception/releases/download/inception-23.0.1/inception-app-webapp-23.0.1-standalone.jar -O /srv/inception/inception.jar
sudo chown www-data:www-data /srv/inception/inception.jar
sudo chmod +x /srv/inception/inception.jar
ls -l /srv/inception

cat << EOF | sudo tee /srv/inception/inception.conf
JAVA_OPTS="-Djava.awt.headless=true -Dinception.home=/srv/inception"
EOF
cat << EOF | sudo tee /etc/systemd/system/inception.service
  [Unit]
  Description=INCEpTION

  [Service]
  ExecStart=/usr/lib/jvm/java-17-openjdk-amd64/bin/java -Djava.awt.headless=true -Dinception.home=/srv/inception -jar /srv/inception/inception.jar
  User=www-data

  [Install]
  WantedBy=multi-user.target
EOF
sudo systemctl disable inception
sudo systemctl enable inception
sudo systemctl daemon-reload

## Start the service
sudo systemctl start inception
sudo systemctl status inception

## Look to the logs
sudo journalctl -u inception
sudo journalctl --rotate
sudo journalctl --vacuum-time=1s

sudo systemctl stop inception
sudo systemctl start inception
sudo service apache2 restart
```
