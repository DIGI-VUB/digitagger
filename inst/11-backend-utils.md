########################################################################################################
## BACKEND
## 
########################################################################################################

########################################################################################################
## NETWORK
## 

```{bash}
sudo apt-get update
sudo apt-get install -y bmon slurm tcptrack
bmon
```

########################################################################################################
## APACHE
## 

```{bash}
sudo apt install -y apache2
sudo apt install -y libapache2-mod-proxy-html
sudo apt install -y libxml2-dev
sudo apt install -y libapache2-mod-auth-openidc
wget https://github.com/zmartzone/mod_auth_openidc/releases/download/v2.4.11.1/libapache2-mod-auth-openidc_2.4.11.1-1.focal+1_amd64.deb
sudo dpkg -i libapache2-mod-auth-openidc_2.4.11.1-1.focal+1_amd64.deb
rm libapache2-mod-auth-openidc_2.4.11.1-1.focal+1_amd64.deb
service apache2 status
sudo a2enmod ssl
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo apachectl restart
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_wstunnel
sudo a2enmod headers
sudo a2enmod auth_openidc
sudo a2enmod proxy proxy_ajp
sudo a2enmod dump_io
sudo service apache2 restart 
sudo apachectl -M 
sudo lsof -i -P -n | grep LISTEN
```

########################################################################################################
## DOCKER
##

```{bash}
sudo groupadd docker
echo $USER
sudo gpasswd -a $USER docker
sudo apt install -y docker-compose
sudo apt install -y docker.io
sudo service docker restart
sudo chmod 666 /var/run/docker.sock
```


########################################################################################################
## Unzip
##

```{bash}
sudo apt install -y unzip
```