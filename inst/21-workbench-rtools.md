########################################################################################################
## WORKBENCH
## 
########################################################################################################


####################################################
## R
## 

```{bash}
sudo apt update
sudo apt upgrade
sudo apt -y autoremove
sudo apt install -y r-base r-base-dev
```

- upgrade R to latest version 

```{bash}
sudo apt update -qq
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
sudo add-apt-repository ppa:c2d4u.team/c2d4u4.0+
sudo apt upgrade
```

########################################################################################################
## RStudio
## 

```{bash}
sudo apt install -y gdebi-core
wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-2022.02.0-443-amd64.deb
sudo gdebi -n rstudio-server-2022.02.0-443-amd64.deb
rm rstudio-server-2022.02.0-443-amd64.deb
sudo service rstudio-server status
sudo journalctl -u rstudio-server.service
```

########################################################################################################
## SHINY SERVER
## 

```{bash}
sudo R -e "install.packages('shiny', repos = 'https://cloud.r-project.org/')"
sudo R -e "install.packages('rmarkdown', repos = 'https://cloud.r-project.org/')"

wget https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.17.973-amd64.deb
sudo gdebi -n shiny-server-1.5.17.973-amd64.deb
rm shiny-server-1.5.17.973-amd64.deb
sudo systemctl status shiny-server
sudo service shiny-server status
```


########################################################################################################
## R package dependencies
## 

```{bash}
sudo apt update
sudo apt install -y libxml2-dev
sudo apt install -y libcurl4-openssl-dev
sudo apt install -y libmagick++-dev
sudo apt install -y libudunits2-dev
sudo apt install -y libgdal-dev
sudo apt install -y libpq-dev
```