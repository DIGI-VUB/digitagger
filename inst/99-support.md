
## Documentation

- https://hpcugent.github.io/vsc_user_docs/pdf/intro-Cloud.pdf
- Inspect GPU/CPU

```
sudo lspci -v | less
nvidia-smi
```

Instances should use the 
- _vm network for communication
- the _nfs network if they need access to shared file systems (see chapter 9). 
- On the other hand _vsc network is used to connect to or provide VSC services via VSC network and floating IPs



##

- VSC Tier-1
    * 1 public IP address for public network access
    * 300 GB of storage on the shared filesystem of the VSC Cloud
    * 1 CPUv1 VM and 1 GPUv1 VM, 
        - for a total of 4 vCPUs and 1 vGPUs, 
        - 128 GB of RAM and a 
        - persistent local disk space of 1 TB (in accordance with the request in your proposal) for the duration of the project.

- Questions
    * The coupling with other VSC components, Tier-2, Tier-1 Compute and Tier-1 DATA is mentioned, but no details are given. Could you clarify how you would couple with these components?
    * We suspect that UPSv1 (instead of CPUv1) flavour VMs are needed for your project. A perceived risk is that if you host a database on a non-UPS supported VM, this could lead to corruption in the event of a power failure. UPSv1 VMs are supported by an uninterruptible power source.
      - CPUv1 VMs are not supported by an UPS
    * What authentication method will be used for the applications?

- **Containers**: https://github.com/gjbex/Containers-for-HPC

Contact cloud@vscentrum.be


- Single-sign on: https://idp.vub.be/idp/profile/Authn/SAML2/POST/SSO

- Backup GPU

```
sudo su digi
bash
sudo mkdir /mnt/digishare/digi
sudo cp -R /home/digi/* /mnt/digishare/digi
```



