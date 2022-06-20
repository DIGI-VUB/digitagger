########################################################################################################
## WORKBENCH
## 
########################################################################################################


####################################################
## R
## 

- Install R

```{bash}
sudo apt update
sudo apt install -y r-base r-base-dev
```

- upgrade R to latest version 

```{bash}
sudo apt update -qq
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
sudo add-apt-repository ppa:c2d4u.team/c2d4u4.0+
sudo apt update
sudo apt list --upgradable
sudo apt install -y r-base r-base-dev r-base-html r-doc-html
sudo apt list --upgradable
```

-  Use https://eddelbuettel.github.io/r2u/

```{bash}
wget -q -O- https://eddelbuettel.github.io/r2u/assets/dirk_eddelbuettel_key.asc | sudo tee -a /etc/apt/trusted.gpg.d/cranapt_key.asc
echo "deb [arch=amd64] https://dirk.eddelbuettel.com/cranapt focal main" | sudo tee -a /etc/apt/sources.list.d/cranapt.list
sudo apt update
sudo apt list --upgradable
sudo apt install -y r-cran-class r-cran-boot r-cran-class r-cran-cluster r-cran-codetools r-cran-foreign r-cran-kernsmooth r-cran-mass r-cran-matrix r-cran-mgcv r-cran-nlme r-cran-nnet r-cran-rpart r-cran-spatial r-cran-survival
sudo apt list --upgradable
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
sudo apt install -y r-cran-shiny r-cran-rmarkdown 

wget https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.17.973-amd64.deb
sudo gdebi -n shiny-server-1.5.17.973-amd64.deb
rm shiny-server-1.5.17.973-amd64.deb
sudo systemctl status shiny-server
sudo service shiny-server status
```

########################################################################################################
## RETICULATE
## 

```{bash}
sudo apt install -y r-cran-reticulate
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