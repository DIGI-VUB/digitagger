########################################################################################################
## CERTBOT SSL
##

```{bash}
sudo apt install -y certbot python3-certbot python3-certbot-apache
```

## SSL certificates

```{bash}
## Execute the following (wildcard subdomain certificate for digitagger.org) + add the DNS TXT record with name `_acme-challenge.digitagger.org` to the DNS entry 
sudo certbot certonly --apache --preferred-challenges=dns --email jan.wijffels@vub.be --agree-tos -d *.digitagger.org
```

- For the SSL certificate of mailserver.digitagger.org only

```{bash}
cat << EOF | sudo tee  /etc/apache2/sites-available/mailserver.conf
<VirtualHost *:443>
    ServerName mailserver.digitagger.org 
    ServerAlias www.mailserver.digitagger.org 
    DocumentRoot /var/www/html
</VirtualHost>
EOF
sudo a2dissite mailserver
sudo a2ensite mailserver
sudo apache2ctl configtest
sudo service apache2 restart

sudo certbot certonly -d mailserver.digitagger.org --apache --dry-run
sudo certbot certonly -d mailserver.digitagger.org --apache
sudo a2dissite mailserver
```

########################################################################################################
## Ports
##

```{bash}
sudo lsof -i -P -n | grep LISTEN
sudo ss -tulpn | grep LISTEN
sudo lsof -i:22
```

########################################################################################################
################################### APACHE CONFIG ######################################################
########################################################################################################

########################################################################################################
## KEYCLOAK CONFIG
##

```{bash}
cat << EOF | sudo tee /etc/apache2/sites-available/keycloak.conf
<VirtualHost *:443>
    ServerName iam.digitagger.org 
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem
    ProxyPreserveHost On
    ProxyRequests Off
    SSLProxyEngine On
    SSLProxyCheckPeerCN on
    SSLProxyCheckPeerExpire on
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"
    ProxyPass / http://127.0.0.1:28080/
    ProxyPassReverse / http://127.0.0.1:28080/
</VirtualHost>
EOF
sudo a2dissite keycloak
sudo a2ensite keycloak
sudo apache2ctl configtest
sudo service apache2 restart
```



########################################################################################################
## RSTUDIO SERVER CONFIG
##

```{bash}
cat << EOF | sudo tee  /etc/apache2/sites-available/rstudio.conf
<VirtualHost *:443>
    ServerName rstudio.digitagger.org
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem
    ## Keycloak login
    OIDCProviderMetadataURL https://iam.digitagger.org/auth/realms/dev/.well-known/openid-configuration
    OIDCRedirectURI https://rstudio.digitagger.org/index.html
    OIDCClientSecret $KEYCLOAK_CLIENT_RSTUDIO_SECRET
    OIDCCryptoPassphrase $KEYCLOAK_CLIENT_CRYPTO
    OIDCClientID rstudio-client
    # Keep sessions alive for 10 hours
    OIDCSessionInactivityTimeout 36000
    OIDCSessionMaxDuration 36000    
    <Location />
      AuthType openid-connect
      Require valid-user
      LogLevel debug
    </Location>
    ## Proxy setup
    <Proxy *>
      Allow from localhost
    </Proxy>
    ProxyPass / http://$IP_WERKBANK:8787/
    ProxyPassReverse / http://$IP_WERKBANK:8787/
    ProxyPreserveHost On
    RequestHeader set X-Forwarded-Proto https
</VirtualHost>
EOF
sudo a2dissite rstudio
sudo a2ensite rstudio
sudo apache2ctl configtest
sudo service apache2 restart
```

########################################################################################################
## SHINY SERVER CONFIG
##

```{bash}
cat << EOF | sudo tee /etc/apache2/sites-available/shiny.conf
<VirtualHost *:443>
    ServerName apps.digitagger.org 
    ServerAlias www.apps.digitagger.org 
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem
    ## Keycloak login
    OIDCProviderMetadataURL https://iam.digitagger.org/auth/realms/dev/.well-known/openid-configuration
    OIDCRedirectURI https://apps.digitagger.org/secure.html
    OIDCClientSecret $KEYCLOAK_CLIENT_SHINY_SECRET
    OIDCCryptoPassphrase $KEYCLOAK_CLIENT_CRYPTO
    OIDCClientID shiny-client
    # Keep sessions alive for 10 hours
    OIDCSessionInactivityTimeout 36000
    OIDCSessionMaxDuration 36000    
    # Pass on user information in the header    
    OIDCRemoteUserClaim preferred_username
    OIDCAuthNHeader remote_user
    OIDCInfoHook userinfo
    ## TODO: define a mapping like https://apps.digitagger.org/user/xyz
    #<Location />
    #  AuthType openid-connect
    #  Require valid-user
    #  LogLevel debug
    #</Location>
    ## Proxy setup
    <Proxy *>
      Allow from localhost
    </Proxy>
    RequestHeader set X-Forwarded-Proto https
    ProxyPass / http://$IP_WERKBANK:3838/
    ProxyPassReverse / http://$IP_WERKBANK:3838/
</VirtualHost>
EOF
sudo a2dissite shiny
sudo a2ensite shiny
sudo apache2ctl configtest
sudo service apache2 restart
```

########################################################################################################
## INCEPTION CONFIG
##

```
cat << EOF | sudo tee /etc/apache2/sites-available/inception.conf
<VirtualHost *:443>
    ServerName inception.digitagger.org 
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem
    ## Keycloak login
    OIDCProviderMetadataURL https://iam.digitagger.org/auth/realms/dev/.well-known/openid-configuration
    OIDCRedirectURI https://inception.digitagger.org/secure.html
    OIDCClientSecret $KEYCLOAK_CLIENT_INCEPTION_SECRET
    OIDCCryptoPassphrase $KEYCLOAK_CLIENT_CRYPTO
    OIDCClientID inception-client
    # Keep sessions alive for 10 hours
    OIDCSessionInactivityTimeout 36000
    OIDCSessionMaxDuration 36000    
    # Pass on user information in the header    
    OIDCRemoteUserClaim preferred_username
    OIDCAuthNHeader remote_user
    OIDCInfoHook userinfo    
    <Location />
      AuthType openid-connect
      Require valid-user
      LogLevel debug
    </Location>
    ProxyRequests Off
    ProxyPreserveHost Off
    ProxyPass / http://localhost:18080/
    ProxyPassReverse / http://localhost:18080/
</VirtualHost>
EOF
sudo a2dissite inception
sudo a2ensite inception
sudo apachectl configtest
sudo service apache2 restart
```

########################################################################################################
## LOOK TO THE LOGS
##

```{bash}
sudo tail /var/log/apache2/access.log -n 100
sudo tail /var/log/apache2/other_vhosts_access.log -n 100
sudo tail /var/log/apache2/error.log -n 100
sudo cat /var/log/apache2/error.log
cp /var/log/apache2/error.log error.log
sudo truncate -s 0 /var/log/apache2/access.log
sudo truncate -s 0 /var/log/apache2/other_vhosts_access.log
sudo truncate -s 0 /var/log/apache2/error.log
```

