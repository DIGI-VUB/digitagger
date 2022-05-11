####################################################
## Users
## 

```
sudo useradd digi --create-home --password $DIGI_PWD
sudo passwd --status digi
sudo passwd digi

sudo su digi
bash
cd /home/digi
git config --global user.email "jan.wijffels@vub.be"
git config --global user.name "Jan Wijffels"
```