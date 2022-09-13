## Mounting volumes

- digidb (volume which can be attached to 1 instance): volume attached to `digitagger` containing DB 
    - note: `sudo mkfs.ext4 /dev/vdb` erases the disk

```
## Format the disk to Linux ext4 - first time only
df -h --output=source,avail,pcent,target
sudo fdisk -l
sudo mkfs.ext4 /dev/vdb # NOTE: this erases the disk
sudo fdisk -l
## Mount the specific location
sudo mkdir --parents /mnt/digidb
sudo mount -t ext4 /dev/vdb /mnt/digidb
mount
## Cron job at reboot
@reboot mount -t ext4 /dev/vdb /mnt/digidb
```

- digidisk (volume which can be attached to 1 instance): volume attached to `digiwerkbank` containing DB and user folders 
    - note: `sudo mkfs.ext4 /dev/vdb` erases the disk

```
## Format the disk to Linux ext4 - first time only
df -h --output=source,avail,pcent,target
sudo fdisk -l
sudo mkfs.ext4 /dev/vdb # NOTE: this erases the disk
sudo fdisk -l
## Mount the specific location
sudo mkdir --parents /mnt/digidisk
sudo mount -t ext4 /dev/vdb /mnt/digidisk
mount
sudo su root
mkdir /mnt/digidisk/home
ls -l /mnt/digidisk/home
## Cron job at reboot sudo crontab -e
@reboot mount -t ext4 /dev/vdb /mnt/digidisk
```

- digishare (NFS shared file system)
    - note: this does not erase the disk
    - should be done on both `digitagger` and `digiwerkbank`

```
sudo apt install -y nfs-common
sudo apt install -y nfs-kernel-server
sudo mkdir --parents /mnt/digishare
sudo mount -t nfs 10.131.35.194:/volumes/_nogroup/873ff0cd-249b-4adf-98c2-3f1ec0452eb2 /mnt/digishare
ls -l /mnt/digishare
ls -l /mnt/digishare/brat/brat/data
## Cron job at reboot sudo crontab -e
@reboot mount -t nfs 10.131.35.194:/volumes/_nogroup/873ff0cd-249b-4adf-98c2-3f1ec0452eb2 /mnt/digishare
```

- allow group developers to add elements to the disk (both as well on GPU / CPU instance)

```
sudo groupadd developers
sudo chown -R ubuntu:developers /mnt/digishare
sudo chown -R ubuntu:developers /mnt/digidb
sudo chown -R ubuntu:developers /mnt/digidisk
sudo usermod -a -G developers digi
sudo usermod -a -G developers ubuntu

# Fix rights where needed
sudo chown -R root:root /mnt/digidisk/home
sudo chown -R shiny:shiny /mnt/digidisk/home/shiny
sudo chown -R digi:digi /mnt/digidisk/home/digi
sudo chown root:root /mnt/digidisk
```

###
### BACKUP AND RECOVER
###

```
sudo cp -r /mnt/digishare/digi/* /mnt/digidisk/home/digi/
sudo chown -R digi:digi /mnt/digidisk/home/digi/
```

## Inspect

```
sudo blkid
sudo service cron status
sudo crontab -e
## at digiwerkbank
@reboot mount -t ext4 /dev/vdb /mnt/digidisk
@reboot mount -t nfs 10.131.35.194:/volumes/_nogroup/873ff0cd-249b-4adf-98c2-3f1ec0452eb2 /mnt/digishare
## at digitagger backend
@reboot mount -t ext4 /dev/vdb /mnt/digidb
@reboot mount -t nfs 10.131.35.194:/volumes/_nogroup/873ff0cd-249b-4adf-98c2-3f1ec0452eb2 /mnt/digishare
```