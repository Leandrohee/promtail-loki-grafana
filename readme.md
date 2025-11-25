# creating loki-data folder before running docker compose up

The loki-data folder has to be create before running the docker using this command 

```bash
mkdir -p loki-data/chunks
mkdir -p loki-data/rules
chmod -R 777 loki-data 
```


# the url to connect a datasource in grafana is 

- http://loki:3100


# Códigos prontos


- Conta quantos usuários estavam logados nos últimos 5 minutos 
```bash
count(
  sum by (user) (
    count_over_time(
      {job="cerberusjwt_job"} |= "lokiLog" | json | user!="user_undefined" [5m]
    )
  )
)
```

- Conta a ultima query executada por um usuário
```bash
sum by (user, graphqlFunction) (
  count_over_time(
    {job="cerberusjwt_job"} |= "lokiLog" | json | systemName = "soucbmdf"| user != "user_undefined" [5m]
  )
)
```

- Quais queries foram executadas no 
```bash
{job="cerberusjwt_job"} |= "lokiLog" | json
```

- Pega todo json que tenha fnSoubombeiroOdontoListaUsuarioSaude na graphlFunction e "soucbmdf" no systemName
```bash
{job="cerberusjwt_job"} | json | graphqlFunction=fnSoubombeiroOdontoListaUsuarioSaude | systemName="soucbmdf"
```

# LogsQl já otimizados


1. Conta quantos usuários estavam logados nos últimos 5 minutos 
```bash
count(
  sum by (user) (
    count_over_time(
      {job="cerberusjwt_job"} |= "lokiLog" | json | user!="user_undefined" [5m]
    )
  )
)
```

2. Pega todos os usarios por tempo - aplique isso com o transform join e options/type_range/5m e tambem options/legends={{user}} - aplique também organize e coloque os usuários em ordem ascendente (transform-organize)
```bash
(sum by (user) 
    (count_over_time(
        {job="cerberusjwt_job"} 
        |= "lokiLog" 
        | json 
        | user != "user_undefined"      
        [5m]
    ))
) 
/
(sum by (user) 
    (count_over_time(
        {job="cerberusjwt_job"} 
        |= "lokiLog" 
        | json 
        | user != "user_undefined"      
        [5m]
    ))
) 
```

3. Pega todo json que tenha "graphqlFunction" dentro. 
  - Table -> Transformation/Extract fields -> source: labels/format: json -> user,graphqlFunction,executedTime,responseBody
  - Cell options -> cell value inspect -> true
  - Cell options ->

```bash
{job="cerberusjwt_job"} | json |= "lokiLog" | systemName="soucbmdf"
```

4. Pega todas as queries executadas em 1 minutoes

queries -> options: type: instant (Gauge)
queries -> options: type: Range ( time serie)


```bash
sum(
  count_over_time(
    {job="cerberusjwt_job"} | json |= `lokiLog` | systemName = `soucbmdf` [1m]
  )
)
```