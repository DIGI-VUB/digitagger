########################################################################################################
## Installation procedure for BRAT
##

```{bash}
sudo mkdir $BRAT_HOME
sudo chown -R ubuntu:developers $BRAT_HOME
cd $BRAT_HOME
sudo git clone https://github.com/nlplab/brat
sudo chown -R ubuntu:ubuntu $BRAT_HOME/brat
cd brat
git checkout 44178b632017e989eb0b0c747bedb76e78ce8a39
```

- Install the shell script

```{bash}
./install.sh ## provide $BRAT_PWD
mkdir -p data work
sudo pip install simplejson
```

```{bash}
sudo apt-get update
sudo apt-get install libapache2-mod-fcgid
sudo a2enmod fcgid
sudo a2enmod rewrite
sudo a2enmod cgid
sudo systemctl restart apache2

sudo chown -R ubuntu:developers $BRAT_HOME
sudo chown -R www-data:developers $BRAT_HOME/brat
sudo chown -R www-data:developers $BRAT_HOME/brat/data
sudo chown -R www-data:developers $BRAT_HOME/brat/work
sudo chmod +rwx $BRAT_HOME/brat/data
#sudo chown -R www-data:digi $BRAT_HOME/brat/data/Getuigenissen
#sudo chown -R www-data:digi $BRAT_HOME/brat/data/tryouts
```

```{bash}
cat << EOF | sudo tee /etc/apache2/sites-available/brat.conf
<VirtualHost *:443>
    ServerName brat.digitagger.org 
    ServerAlias www.brat.digitagger.org
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/digitagger.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/digitagger.org/privkey.pem

    DocumentRoot $BRAT_HOME/brat
    DirectoryIndex index.html
    <Directory "$BRAT_HOME/brat">
        AllowOverride Options Indexes FileInfo Limit AuthConfig 
        AddType application/xhtml+xml .xhtml
        AddType font/ttf .ttf
        # For CGI support
        AddHandler cgi-script .cgi
        # Comment out the line above and uncomment the line below for FastCGI
        #AddHandler fastcgi-script fcgi
        Options +ExecCGI +FollowSymLinks
        DirectoryIndex index.xhtml
        Require all granted
        <FilesMatch "^(\.htaccess|config\.py)$">
          Require all denied
        </FilesMatch>
    </Directory>
</VirtualHost>
EOF
sudo a2dissite brat
sudo a2ensite brat
sudo apache2ctl configtest
sudo service apache2 restart

cat /var/log/apache2/error.log
```

- COPY Getuigenissen
