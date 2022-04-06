########################################################################################################
## Mail server
## 

- docker compose script will be stored at /inst/mailserver
- mailserver information at the volume at $DIGIDB_MAILSERVER

```{bash}
mkdir --parents $DIGITAGGER_HOME/inst/mailserver
mkdir --parents $DIGIDB_MAILSERVER
```

- Setup user / password of email

```
## Create user and docker-data/dms folder
echo $POSTFIX_PWD  
sudo rm -r $DIGIDB_MAILSERVER/docker-data
docker run --rm -v "${DIGIDB_MAILSERVER}/docker-data/dms/config/:/tmp/docker-mailserver/" docker.io/mailserver/docker-mailserver:10.5.0 setup email add info@digitagger.org "${POSTFIX_PWD}"
docker run --rm -v "${DIGIDB_MAILSERVER}/docker-data/dms/config/:/tmp/docker-mailserver/" docker.io/mailserver/docker-mailserver:10.5.0 setup alias add postmaster@digitagger.org info@digitagger.org
docker run --rm -v "${DIGIDB_MAILSERVER}/docker-data/dms/config/:/tmp/docker-mailserver/" docker.io/mailserver/docker-mailserver:10.5.0 setup email list
docker run --rm -v "${DIGIDB_MAILSERVER}/docker-data/dms/config/:/tmp/docker-mailserver/" docker.io/mailserver/docker-mailserver:10.5.0 setup alias list
#docker run --rm -v "${DIGIDB_MAILSERVER}/docker-data/dms/config/:/tmp/docker-mailserver/" docker.io/mailserver/docker-mailserver:10.5.0 setup email del info@digitagger.org
```

- add in DNS MX / TXT record (see https://docker-mailserver.github.io/docker-mailserver/edge/config/best-practices/spf/)
    - MX: `digitagger.org. IN  MX 1 mailserver.digitagger.org.`
    - TXT: `digitagger.org. IN TXT "v=spf1 mx ~all"`


## docker-compose file

```{bash}
cat << EOF | sudo tee $DIGITAGGER_HOME/inst/mailserver/docker-compose.yml
version: '3.3'
services:
  mailserver:
    image: docker.io/mailserver/docker-mailserver:10.5.0
    container_name: clariah-mailserver
    hostname: mailserver.digitagger.org
    ports:
      - "25:25"     # SMTP  (explicit TLS => STARTTLS)
      - "465:465"   # ESMTP (implicit TLS)
      - "587:587"   # ESMTP (explicit TLS => STARTTLS)
    volumes:
      - ${DIGIDB_MAILSERVER}/docker-data/dms/mail-data/:/var/mail/
      - ${DIGIDB_MAILSERVER}/docker-data/dms/mail-state/:/var/mail-state/
      - ${DIGIDB_MAILSERVER}/docker-data/dms/mail-logs/:/var/log/mail/
      - ${DIGIDB_MAILSERVER}/docker-data/dms/config/:/tmp/docker-mailserver/
      - ${DIGIDB_MAILSERVER}/docker-data/certbot/certs/:/etc/letsencrypt/"
      - ${DIGIDB_MAILSERVER}/docker-data/certbot/logs/:/var/log/letsencrypt/" 
      - /etc/localtime:/etc/localtime:ro
      - /etc/letsencrypt:/etc/letsencrypt    
    environment:
      - ENABLE_SPAMASSASSIN=0
      - SPAMASSASSIN_SPAM_TO_INBOX=1
      - ENABLE_CLAMAV=0
      - ENABLE_FAIL2BAN=0
      - ENABLE_POSTGREY=0
      - ENABLE_SASLAUTHD=0
      - ONE_DIR=1
      - DMS_DEBUG=0
      - LOG_LEVEL=info
      - SUPERVISOR_LOGLEVEL=info
      - POSTMASTER_ADDRESS=postmaster@digitagger.org
      - SSL_TYPE=letsencrypt
    cap_add:
      - NET_ADMIN
      - SYS_PTRACE
    restart: always
EOF
```

```{bash}
## Start it up
sudo docker-compose -f $DIGITAGGER_HOME/inst/mailserver/docker-compose.yml up --detach 
sudo docker-compose -f $DIGITAGGER_HOME/inst/mailserver/docker-compose.yml logs
## Pull it down if needed
sudo docker-compose -f $DIGITAGGER_HOME/inst/mailserver/docker-compose.yml down
## Debugging
sudo docker exec -it clariah-mailserver bash
     apt-get update && apt-get install -y vim less
     less /var/log/mail/mail.log
     less /var/log/supervisor/clamav.log
```

- Add a DKIM key and restart

```{bash}
cd $DIGITAGGER_HOME
wget https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/master/setup.sh
chmod a+x ./setup.sh
./setup.sh config dkim
rm setup.sh
#ubuntu@digitagger-prd-cpu:~/digitagger$ ./setup.sh config dkim
#Creating DKIM private key /tmp/docker-mailserver/opendkim/keys/digitagger.org/mail.private
#Creating DKIM KeyTable
#Creating DKIM SigningTable
#Creating DKIM TrustedHosts
sudo docker exec -it clariah-mailserver cat /tmp/docker-mailserver/opendkim/keys/digitagger.org/mail.txt > dkim.txt
cat dkim.txt
## Put content of DKIM key in DNS https://docker-mailserver.github.io/docker-mailserver/edge/config/best-practices/dkim/
## Restart the service
sudo docker-compose -f $DIGITAGGER_HOME/inst/mailserver/docker-compose.yml restart
```
