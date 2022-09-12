########################################################################################################
## CANTALOUPE IIIF SERVER
## 

## Get jar file

- Source code at $DIGITAGGER_HOME/inst/cantaloupe

```{bash}
sudo mkdir /srv/cantaloupe
unzip -l $DIGITAGGER_HOME/inst/cantaloupe/cantaloupe-5.0.5.zip
sudo unzip $DIGITAGGER_HOME/inst/cantaloupe/cantaloupe-5.0.5.zip cantaloupe-5.0.5/cantaloupe-5.0.5.jar -d /srv/cantaloupe
sudo unzip -j $DIGITAGGER_HOME/inst/cantaloupe/cantaloupe-5.0.5.zip cantaloupe-5.0.5/cantaloupe.properties.sample -d $DIGITAGGER_HOME/inst/cantaloupe
```

## Setup configuration

- Configure setup + launch the app
- Configured as BasicLookupStrategy

```{bash}
export CANTALOUPE_PROPERTIES_FILE=$DIGITAGGER_HOME/inst/cantaloupe/cantaloupe.properties
echo $CANTALOUPE_IMAGE_PATH
echo $CANTALOUPE_PROPERTIES_FILE
cp $DIGITAGGER_HOME/inst/cantaloupe/cantaloupe.properties.sample $CANTALOUPE_PROPERTIES_FILE
sed -i "s|source.static = .*|source.static = FilesystemSource|g" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|FilesystemSource.BasicLookupStrategy.path_prefix = /home/myself/images/|FilesystemSource.BasicLookupStrategy.path_prefix = $CANTALOUPE_IMAGE_PATH|" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|endpoint.admin.enabled = false|endpoint.admin.enabled = true|" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|endpoint.admin.username = .*|endpoint.admin.username = $CANTALOUPE_ADMIN_USER|g" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|endpoint.admin.secret =|endpoint.admin.secret = $CANTALOUPE_ADMIN_PWD|" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|overlays.BasicStrategy.enabled = .*|overlays.BasicStrategy.enabled = true|g" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|overlays.BasicStrategy.type = .*|overlays.BasicStrategy.type = string|g" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|overlays.BasicStrategy.string = .*|overlays.BasicStrategy.string = VUB - DIGI|g" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|overlays.BasicStrategy.string.color = .*|overlays.BasicStrategy.string.color = white|g" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|overlays.BasicStrategy.position = .*|overlays.BasicStrategy.position = bottom left|g" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|overlays.BasicStrategy.inset = .*|overlays.BasicStrategy.inset = 10|g" $CANTALOUPE_PROPERTIES_FILE
sed -i "s|slash_substitute.*|slash_substitute = :|g" $CANTALOUPE_PROPERTIES_FILE 
#java -Dcantaloupe.config=cantaloupe.properties -Xmx2g -jar cantaloupe-5.0.5.jar
sudo cp $CANTALOUPE_PROPERTIES_FILE /srv/cantaloupe/cantaloupe-5.0.5/cantaloupe.properties
```

## Define service

- Set up cantaloupe as a service

```{bash}
cat << EOF | sudo tee /etc/systemd/system/cantaloupe.service
  [Unit]
  Description=Cantaloupe

  [Service]
  ExecStart=/usr/lib/jvm/java-17-openjdk-amd64/bin/java -Djava.awt.headless=true -Dcantaloupe.config=/srv/cantaloupe/cantaloupe-5.0.5/cantaloupe.properties -Xmx4g -jar /srv/cantaloupe/cantaloupe-5.0.5/cantaloupe-5.0.5.jar
  User=ubuntu

  [Install]
  WantedBy=multi-user.target
EOF
sudo systemctl disable cantaloupe
sudo systemctl enable cantaloupe
sudo systemctl daemon-reload

## Start the service
sudo systemctl start cantaloupe
sudo systemctl stop cantaloupe
sudo systemctl restart cantaloupe
sudo systemctl status cantaloupe
sudo journalctl -u cantaloupe
sudo journalctl --rotate
sudo journalctl --vacuum-time=1s
```

## Specific for the server itself

- Location of images at /mnt/digishare/iiif/images and  /home/ubuntu/iiif/images points to there

```{bash}
sudo mkdir /mnt/digishare/iiif
sudo chown ubuntu:ubuntu /mnt/digishare/iiif
mkdir /mnt/digishare/iiif/images
mkdir /home/ubuntu/iiif
ln -s /mnt/digishare/iiif/images /home/ubuntu/iiif/images
```

- Upload images in /home/ubuntu/iiif/images

```
cp /mnt/digishare/irods/irodsfuse/vsc10311/example.png /mnt/digishare/iiif/images/
mkdir /mnt/digishare/iiif/images/voorbeeld
cp /mnt/digishare/irods/irodsfuse/vsc10311/example.png /mnt/digishare/iiif/images/voorbeeld/imagexyz.png
```

- Examples
    - example.png at main folder
        - https://iiif.digitagger.org/iiif/3/example.png/info.json
        - https://iiif.digitagger.org/iiif/3/example.png/10,40,100,30/max/0/default.png
        - https://iiif.digitagger.org/iiif/3/example.png/full/max/0/default.png
    - imagexyz.png in voorbeeld subfolder 
        - https://iiif.digitagger.org/iiif/3/voorbeeld:imagexyz.png/info.json
        - https://iiif.digitagger.org/iiif/3/voorbeeld:imagexyz.png/full/max/0/default.png        
    - imagexyz.jpg in voorbeeld subfolder 
        - https://iiif.digitagger.org/iiif/3/voorbeeld:imagexyz.jpg/info.json
        - https://iiif.digitagger.org/iiif/3/voorbeeld:imagexyz.jpg/full/max/0/default.png
    - imagexyz.jpg in voorbeeld subfolder 
        - https://iiif.digitagger.org/iiif/3/getuigenissen:brugse-vrije:RABrugge_I15_17015_V16_02.jpg/info.json  
        - https://iiif.digitagger.org/iiif/3/getuigenissen:brugse-vrije:RABrugge_I15_17015_V16_02.jpg/10,40,100,30/max/0/default.png
        - https://iiif.digitagger.org/iiif/3/getuigenissen:brugse-vrije:RABrugge_I15_17015_V16_02.jpg/full/max/0/default.png
        - https://iiif.digitagger.org/iiif/3/getuigenissen:brugse-vrije:RABrugge_I15_17038_V9_11.jpg/full/max/0/default.png
        


