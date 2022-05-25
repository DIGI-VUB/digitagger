# digitagger

Work in progress

- Cantaloupe: https://iiif.digitagger.org
- Inception: https://inception.digitagger.org
- Apps Shiny: https://apps.digitagger.org/home/
- RStudio: https://rstudio.digitagger.org
- Keycloak: https://iam.digitagger.org
- BRAT: https://brat.digitagger.org

### Example Shiny apps

- Open apps
    - https://apps.digitagger.org/digi/open/getuigenissen-search
    - https://apps.digitagger.org/digi/open/getuigenissen2-search
    - https://apps.digitagger.org/digi/open/getuigenissen/getuigenissen-geocoding.Rmd
- Apps where a login is required (and user needs to be part of the group e.g digi/xyz, digi/test)
    - https://apps.digitagger.org/digi/xyz
    - https://apps.digitagger.org/digi/test

### Authentication

- Keycloak and RStudio are configured with 2FA
- Inception is configured with proxy-based authentication through Keycloak
- Shiny applications
    - can be put at /home/userxyz/Shinyapps/open
    - can be put at /home/userxyz/Shinyapps/projectxyz which require you to belong to group userxyz/projectxyz in Keycloak to be able to access it
- Cantaloupe is open accessible for IIIF, the `/admin` URI uses the Cantaloupe login configuration
