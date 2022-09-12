########################################################################################################
## PostgreSQL
## 

```{bash}
sudo apt install -y postgresql-12
sudo apt install -y postgresql-client
sudo apt install -y postgresql-client-common
sudo apt install -y odbc-postgresql
sudo apt install -y unixodbc unixodbc-dev
```

```{r}
library(RPostgreSQL)
con <- dbConnect(odbc::odbc(), 
                 .connection_string = sprintf("Driver={PostgreSQL Unicode};Server=193.190.80.20;Port=55432;Database=digi;Uid=%s;Pwd=%s", 
                 Sys.getenv("POSTGRESQL_DIGIVUB_USER"),
                 Sys.getenv("POSTGRESQL_DIGIVUB_PWD")), timeout = 10)
dbListTables(con)
```