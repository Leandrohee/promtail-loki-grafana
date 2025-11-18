# creating loki-data folder before running docker compose up

The loki-data folder has to be create before running the docker using this command 

```bash
mkdir -p loki-data/chunks
mkdir -p loki-data/rules
chmod -R 777 loki-data 
```


# the url to connect a datasource in grafana is 

- http://loki:3100