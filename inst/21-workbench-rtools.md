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
