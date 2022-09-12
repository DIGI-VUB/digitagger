# Connect to Vlaamse Supercomputer

## Directly using SSH key

- Server UI access: https://cloud.vscentrum.be
- IP addresses
    - Public IP machine 193.190.80.20 (this is a public IP in the _vm network range 193.190.80.0/25) 
    - Private IP machines: 172.24.49.33, 172.24.49.34, 172.24.49.35 (these are private IP's in the _vsc network which have addresses 172.24.48.0/20)
- When an instance is created in OpenStack and connected to the _vm, _nfs or _vsc networks, 
    - it is automatically assigned a fixed IP address in that network. 
    - this IP address is permanently associated with the instance until the instance is terminated.
    - Network IP's for project VSC_2021_011 :
        - VSC_2021_011_vm: 10.113.18.51  (from VSC_2021_011_vm_pool  subnet 10.113.18.0/24)
        - VSC_2021_011_vsc: 10.97.8.206  (from VSC_2021_011_vsc_pool subnet 10.97.8.0/24)
        - VSC_2021_011_nfs_vxlan 10.122.133.113
          VSC_2021_011_vsc 10.97.8.33
          VSC_2021_011_vm 10.113.18.61
        - WERKBANK: 10.113.18.7, DB: 10.113.18.210, EMAIL IP: 193.190.80.1


```
## WORKBENCH connect to digitagger-workbench rstudio server running Ubuntu with vGPU
ssh -i D:/Jan/Dropbox/Work/2020/VUB/admin/vsc-private-public-keys/vsc-hydra-openssh-privatekey -p 52001 ubuntu@193.190.80.20
ssh -i D:/Jan/Dropbox/Work/2020/VUB/admin/vsc-private-public-keys/vsc-hydra-openssh-privatekey -p 52001 digi@193.190.80.20
ssh -i D:/Jan/Dropbox/Work/2020/VUB/admin/vsc-private-public-keys/vsc-hydra-openssh-privatekey -p 52001 centos@193.190.80.20
## BACKEND connect to digitagger-prd server running Ubuntu with CPU
ssh -i D:/Jan/Dropbox/Work/2020/VUB/admin/vsc-private-public-keys/vsc-hydra-openssh-privatekey -p 52000 ubuntu@193.190.80.20
## connect to test  server running Ubuntu with CPU
ssh -i D:/Jan/Dropbox/Work/2020/VUB/admin/vsc-private-public-keys/vsc-hydra-openssh-privatekey -p 52002 ubuntu@193.190.80.20
# remove key in case of different port forwarding or in case of new instance
ssh-keygen -R [193.190.80.20]:52000
ssh-keygen -R [193.190.80.20]:52001
ssh-keygen -R [193.190.80.20]:52002
```

## Make sure port forwarding is set up

- Port forwarding

```
ssh -i D:/Jan/Dropbox/Work/2020/VUB/admin/vsc-private-public-keys/vsc-hydra-openssh-privatekey vsc10311@login.hpc.ugent.be

cat << EOF | tee portforwarding.ini
[DEFAULT]
floatingip=193.190.80.20
network=VSC_2021_011_vm

# For instance digitagger
[digitagger]
pattern=digitagger
22=52000
80=80
443=443
25=50025
465=50465
587=50587
5432=55432

# For instance digiwerkbank
[digiwerkbank]
pattern=digiwerkbank
22=52001

# For instance test
[test]
pattern=test
22=52002
EOF

neutron_port_forward --openrc=$HOME/app-cred-jwijffels-openrc.sh --map=$HOME/portforwarding.ini
neutron_port_forward --openrc=$HOME/app-cred-jwijffels-openrc.sh --map=$HOME/portforwarding.ini --show --id
neutron_port_forward --openrc=$HOME/app-cred-jwijffels-openrc.sh --show --id
neutron_port_forward --openrc=$HOME/app-cred-jwijffels-openrc.sh --remove=dcc09906-fa7a-4f4b-a9b6-d41c34685197
neutron_port_forward --openrc=$HOME/app-cred-jwijffels-openrc.sh --remove=3dfe2229-787b-40d2-bdb3-92345a72e8f1,b8ee6c84-0a6a-4d2b-aabe-b5e7265e6ed5,b07f5470-46c1-4613-afef-bb5f8c2747d0,e68ce71a-8fce-4186-beca-eadc4047abab,942b21a2-214f-4fdd-b639-9ff239d1364c,1d683053-6146-4a24-9a0a-4f65337e6475,11d03699-de9f-43c1-a5cf-1f01c24db6ff,00ad5fa0-076b-4c29-941a-dd895d483359,373eb55a-6b96-4e62-ace0-35a8b91b39e0,6a411f5e-7430-4041-9463-6f33ba716c52

## Access using SSH or go to Apache on port 52080
# ssh -i $HOME/.ssh/vsc-hydra-openssh-privatekey -p 52000 ubuntu@193.190.80.20
# 193.190.80.20:52080
```


## From Hydra

- VSC_DATA is for me    `/data/brussel/103/vsc10311` and has subfolder: `/data/brussel/103/vsc10311/getuigenissen/data`
- VSC_SCRATCH is for me `/scratch/brussel/103/vsc10311`

```
ssh -i D:/Jan/Dropbox/Work/2020/VUB/admin/vsc-private-public-keys/vsc-hydra-openssh-privatekey vsc10311@login.hpc.vub.be
pwd
echo $VSC_DATA 
echo $VSC_SCRATCH 
cd $VSC_DATA
ls -l
cd $VSC_SCRATCH
ls -l
```

- SSH key

```
ls -l $HOME/.ssh/vsc-hydra-openssh-privatekey 
chmod 600 $HOME/.ssh/vsc-hydra-openssh-privatekey 
```

## Resize the machines 

- openstack from the command line: source credentials and resize the machines

```
source $HOME/app-cred-jwijffels-openrc.sh
openstack server list
openstack server resize --flavor GPUv1.2xlarge 1a9627c2-6e49-41ae-8707-0f676a943877 
openstack server resize --flavor CPUv1.large	 0b7c0167-01c4-45d9-a0dc-c9b0c7ebaef6
```

## Other openstack commands

- These are just to test

```
source $HOME/app-cred-jwijffels-openrc.sh
openstack flavor list
openstack image list
openstack server list
openstack server show digiwerkbank
openstack image show digitagger-prd
openstack security group list
openstack security group rule --help
openstack security group rule list --help
openstack security group rule create --help
openstack security group rule list webports --long
openstack security group rule list werkbanktools
openstack security group show 841e4ad7-0a42-429e-872c-d72b54487090 --fit
openstack security group show b95c865b-d577-44c5-ad93-ffa0d13c1415 --fit
openstack security group rule create webports --ingress --protocol tcp  --remote-ip 0.0.0.0/0 --description "Flask"
openstack security group rule create webports --ingress --protocol tcp  --remote-ip 0.0.0.0/0 --description "Flask"
openstack security group rule create werkbanktools --ingress --protocol tcp --dst-port 5000:5000   --remote-ip 0.0.0.0/0 --description "Flask"
openstack security group rule create werkbanktools --ingress --protocol tcp --dst-port 50000:50000 --remote-ip 0.0.0.0/0 --description "Flask"
openstack security group rule create werkbanktools --ingress --protocol tcp --dst-port 50001:50001 --remote-ip 0.0.0.0/0 --description "Flask"
openstack security group rule create test --ingress --protocol any  
openstack security group rule create test --ingress --protocol any  
openstack security group rule create test --egress --protocol any  
```