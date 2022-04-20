#############################################################################
## Shiny server configuration
##

- Shiny config at `/etc/shiny-server/shiny-server.conf`

```
cat << EOF | sudo tee /etc/shiny-server/shiny-server.conf
# applications with user_dirs run as that user, others default to shiny if not specified
run_as :HOME_USER: shiny;
server {
  listen 3838;
  ##
  ## Default shiny apps - it works 
  ##
  location /home {
    site_dir /srv/shiny-server;
    log_dir /var/log/shiny-server;
    directory_index on;
  }
  ##
  ## User Shiny apps running under /home/userxyz/ShinyApps
  ##
  location / {
    user_apps;
    directory_index on;
  }
}
EOF
```

- Restart the Shiny server when updating the configuration as follows

```
sudo service shiny-server restart
sudo systemctl status shiny-server
```
