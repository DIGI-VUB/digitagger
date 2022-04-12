########################################################################################################
## iRODS 
##   - iRODS client
##   - iRODS fuse (https://github.com/cyverse/irodsfs)
##

## Install iRODS client

- Install iRODS client 

```
LSB_RELEASE="bionic"
wget -qO - https://packages.irods.org/irods-signing-key.asc | sudo apt-key add
echo "deb [arch=amd64] https://packages.irods.org/apt/ ${LSB_RELEASE}  main" | sudo tee /etc/apt/sources.list.d/renci-irods.list
sudo apt-get update
```

- Shell and Python client for iRODS

```
wget -c \
  http://security.ubuntu.com/ubuntu/pool/main/p/python-urllib3/python-urllib3_1.22-1ubuntu0.18.04.2_all.deb \
  http://security.ubuntu.com/ubuntu/pool/main/r/requests/python-requests_2.18.4-2ubuntu0.1_all.deb \
  http://security.ubuntu.com/ubuntu/pool/main/o/openssl1.0/libssl1.0.0_1.0.2n-1ubuntu5.8_amd64.deb
sudo apt install -y \
  ./python-urllib3_1.22-1ubuntu0.18.04.2_all.deb \
  ./python-requests_2.18.4-2ubuntu0.1_all.deb \
  ./libssl1.0.0_1.0.2n-1ubuntu5.8_amd64.deb
rm -rf \
  ./python-urllib3_1.22-1ubuntu0.18.04.2_all.deb \
  ./python-requests_2.18.4-2ubuntu0.1_all.deb \
  ./libssl1.0.0_1.0.2n-1ubuntu5.8_amd64.deb
sudo apt install -y irods-icommands
ils
sudo apt install -y python3-pip
pip install python-irodsclient
```

## Setup connection to iRODS at HPC infrastructure at irods.hpc.kuleuven.be

- Import iRODS SSL certficate from irods.hpc.kuleuven.be
    - Go to irods.kuleuven.be / https://irods.hpc.kuleuven.be/auth/login + export as base64 encoded file
    - Install the certificate on the machine

```
sudo apt-get install -y ca-certificates
cat << EOF | sudo tee -a /usr/local/share/ca-certificates/irods_kuleuven.crt
-----BEGIN CERTIFICATE-----
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
-----END CERTIFICATE-----
EOF
sudo update-ca-certificates
```

- Setup connection to iRODS 
  - visit https://irods.hpc.kuleuven.be, get login
  - irods.hpc.kuleuven.be should be used, and should resolve to 172 address there (certainly wherever .vsc-addresses are resolvable)
  - note: irods.tier1.leuven.vsc should no longer be used

```
mkdir -p ~/.irods
cat << EOF | tee ~/.irods/irods_environment.json
{
  "irods_host": "irods.hpc.kuleuven.be",
  "irods_port": 1247,
  "irods_zone_name": "kuleuven_tier1_pilot",
  "irods_authentication_scheme": "PAM",
  "irods_encryption_algorithm": "AES-256-CBC",
  "irods_encryption_salt_size": 8,
  "irods_encryption_key_size": 32,
  "irods_encryption_num_hash_rounds": 16,
  "irods_user_name": "$IRODS_USER",
  "irods_ssl_ca_certificate_file": "",
  "irods_ssl_verify_server": "cert",
  "irods_client_server_negotiation": "request_server_negotiation",
  "irods_client_server_policy": "CS_NEG_REQUIRE",
  "irods_default_resource": "default"
}
EOF
iinit --ttl 336 $IRODS_IIINIT && echo You are now authenticated to irods. Your session is valid for 2 weeks
ils
```

## Install iRODS-FUSE client v.0.6.0

- Install binary

```{bash}
sudo apt install -y golang
sudo apt install -y fuse

wget https://github.com/cyverse/irodsfs/releases/download/v0.6.0/irodsfs_amd64_linux_v0.6.0.tar
sudo tar xf irodsfs_amd64_linux_v0.6.0.tar -C /usr/local/bin
```

- Set up configuration for `cyverse/irodsfs` in order to connect to irods.hpc.kuleuven.be

```{bash}
cat << EOF | tee ~/.irods/config.yaml
host: irods.hpc.kuleuven.be
port: 1247
proxy_user: $IRODS_USER
client_user: $IRODS_USER
zone: kuleuven_tier1_pilot
password: "$IRODS_IIINIT"

authscheme: pam
ssl_ca_cert_file: "/usr/local/share/ca-certificates/irods_kuleuven.crt"
ssl_encryption_key_size: 32
ssl_encryption_algorithm: "AES-256-CBC"
ssl_encryption_salt_size: 8
ssl_encryption_hash_rounds: 16

path_mappings:
  - irods_path: /kuleuven_tier1_pilot/home/vsc10311
    mapping_path: /vsc10311
    resource_type: dir
EOF
```

- Mount the irods fuse folder to $IRODS_FUSE_HOME

```{bash}
irodsfs --help
mkdir --parents $IRODS_FUSE_HOME
irodsfs -config ~/.irods/config.yaml $IRODS_FUSE_HOME
```

## Specific for the server itself

- Location of images

```{bash}
mkdir /home/ubuntu/irods/
ln -s /mnt/digishare/irods/irodsfuse /home/ubuntu/irods/irodsfuse
```