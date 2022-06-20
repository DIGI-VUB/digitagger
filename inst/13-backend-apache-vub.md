########################################################################################################
## - Same as backend-apache but using VUB SSO only
##

## RSTUDIO

```{bash}
cat << EOF | sudo tee  /etc/apache2/sites-available/rstudio.conf
<VirtualHost *:443>
    ServerName rstudio.digitagger.org
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem
    ## Keycloak login
    OIDCProviderMetadataURL https://sts.windows.net/695b7ca8-2da8-4545-a2da-42d03784e585/.well-known/openid-configuration
    OIDCRedirectURI https://rstudio.digitagger.org/index.html
    OIDCClientSecret $KEYCLOAK_CLIENT_VUB_RSTUDIO_SECRET
    OIDCCryptoPassphrase $KEYCLOAK_CLIENT_CRYPTO
    OIDCClientID $KEYCLOAK_CLIENT_VUB_RSTUDIO
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
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} =websocket
    RewriteRule /(.*)     ws://$IP_WERKBANK:8787/\$1  [P,L]
    RewriteCond %{HTTP:Upgrade} !=websocket
    RewriteRule /(.*)     http://$IP_WERKBANK:8787/\$1 [P,L]
    ProxyPass / http://$IP_WERKBANK:8787/
    ProxyPassReverse / http://$IP_WERKBANK:8787/
    ProxyRequests Off     
</VirtualHost>
EOF
sudo a2dissite rstudio
sudo a2ensite rstudio
sudo apache2ctl configtest
sudo service apache2 restart
```

## INCEPTION


```
cat << EOF | sudo tee /etc/apache2/sites-available/inception.conf
<VirtualHost *:443>
    ServerName inception.digitagger.org 
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem
    ## Keycloak login
    OIDCProviderMetadataURL https://sts.windows.net/695b7ca8-2da8-4545-a2da-42d03784e585/.well-known/openid-configuration
    OIDCRedirectURI https://inception.digitagger.org/secure.html
    OIDCClientSecret $KEYCLOAK_CLIENT_VUB_INCEPTION_SECRET
    OIDCCryptoPassphrase $KEYCLOAK_CLIENT_CRYPTO
    OIDCClientID $KEYCLOAK_CLIENT_VUB_INCEPTION
    # Keep sessions alive for 10 hours
    OIDCSessionInactivityTimeout 36000
    OIDCSessionMaxDuration 36000    
    # Pass on user information in the header    
    #OIDCRemoteUserClaim preferred_username
    OIDCRemoteUserClaim upn
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

## SHINY

```{bash}
cat << EOF | sudo tee /etc/apache2/sites-available/shiny.conf
<VirtualHost *:443>
    ServerName apps.digitagger.org 
    ServerAlias www.apps.digitagger.org 
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem
    ## Keycloak login
    OIDCProviderMetadataURL https://sts.windows.net/695b7ca8-2da8-4545-a2da-42d03784e585/.well-known/openid-configuration
    OIDCRedirectURI https://apps.digitagger.org/app/protected
    OIDCClientSecret $KEYCLOAK_CLIENT_VUB_SHINY_SECRET
    OIDCCryptoPassphrase $KEYCLOAK_CLIENT_CRYPTO
    OIDCClientID $KEYCLOAK_CLIENT_VUB_SHINY
    # Keep sessions alive for 10 hours
    OIDCSessionInactivityTimeout 36000
    OIDCSessionMaxDuration 36000    
    # Pass on user information in the header    
    #OIDCRemoteUserClaim preferred_username
    OIDCRemoteUserClaim upn
    OIDCAuthNHeader remote_user
    OIDCInfoHook userinfo
    ## Mapping as follows
    ##  - https://apps.digitagger.org/userxyz/open/    open access
    ##  - https://apps.digitagger.org/userxyz/public/  open access
    ##  - https://apps.digitagger.org/userxyz/appname/ no open access - needs to belong to group userxyz/appname in Keycloak
    ##  - <LocationMatch "^/user/(?<klantnaam>[^/]+)"> 
    <LocationMatch "^/(?<klantnaam>[^/]+)/(?<appnaam>[^/]+)"> 
      #Require claim "groupmembership:%{env:MATCH_KLANTNAAM}/%{env:MATCH_APPNAAM}" 
      Require valid-user
      AuthType openid-connect 
    </LocationMatch>   
    <LocationMatch "^/.+/open/"> 
      Require all granted
      AuthType openid-connect 
    </LocationMatch>  
    <LocationMatch "^/.+/public/"> 
      Require all granted
      AuthType openid-connect 
    </LocationMatch>            
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

## CANTALOUPE

```
cat << EOF | sudo tee /etc/apache2/sites-available/iiif.conf
<VirtualHost *:443>
    ServerName iiif.digitagger.org
    ServerAlias www.iiif.digitagger.org
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem
    ProxyRequests Off
    ProxyPreserveHost Off
    ProxyPass / http://localhost:8182/ nocanon
    ProxyPassReverse / http://localhost:8182/
    ProxyPassReverseCookiePath / /    
    ProxyPassReverseCookieDomain localhost:8182 iiif.digitagger.org
    ProxyPreserveHost on  
    AllowEncodedSlashes NoDecode
    RequestHeader set X-Forwarded-Proto HTTPS
    RequestHeader set X-Forwarded-Port 443
    RequestHeader set X-Forwarded-Path /  
    ## Pass user access to Keycloak for cantaloupe admin only (not for the images)
    OIDCProviderMetadataURL https://sts.windows.net/695b7ca8-2da8-4545-a2da-42d03784e585/.well-known/openid-configuration
    OIDCRedirectURI https://iiif.digitagger.org/admin
    OIDCClientSecret $KEYCLOAK_CLIENT_VUB_CANTALOUPE_SECRET
    OIDCCryptoPassphrase $KEYCLOAK_CLIENT_CRYPTO
    OIDCClientID $KEYCLOAK_CLIENT_VUB_CANTALOUPE
    <LocationMatch /administration>      
      AuthType openid-connect
      Require valid-user
      LogLevel debug
    </LocationMatch>    
</VirtualHost>
EOF
sudo a2dissite iiif
sudo a2ensite iiif
sudo apachectl configtest
sudo service apache2 restart
```
