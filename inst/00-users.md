####################################################
## Users
## 

```
sudo su root
cat /etc/default/useradd
sed -i 's|SHELL=/bin/sh|SHELL=/bin/bash|g' /etc/default/useradd
sed -i 's|# HOME=/home|HOME=/mnt/digidisk/home|g' /etc/default/useradd
sudo useradd digi --create-home --password $DIGI_PWD
sudo passwd --status digi
#sudo passwd digi
```

Set git configuration of user digi + allows ssh using same key as ubuntu

```
sudo cp -R /home/ubuntu/.ssh /mnt/digidisk/home/digi/
sudo chown -R digi:digi /mnt/digidisk/home/digi/

sudo su digi
cd $HOME
git config --global user.email "jan.wijffels@vub.be"
git config --global user.name "Jan Wijffels"
exit
```