# DOCKER  

## What Problem Docker solves?  
- To eliminate "It works on my machine" problem  

## Why this problem needs to be solved?  
- Engineers collaborate and the environment for the app need to be isolated so we can ship the products/apps in any machine.<br>
## How docker helps in development?
- Docker provides isolated consistent environment and creates virtual networks. So, docker enables easy service communication via container names. Docker also allows port mapping to the host for local development.<br>
## Things to remember
### Stateless
- By default, docker is stateless. Meaning, docker does not retain data or state between runs or termination. Each time a container/containers is/are started, they will start in a fresh instance with no memory of previous condition unless explicitly told to retain by VOLUME MAPPING or (bind mounting).

### Volume Mapping or bind mounting in docker
Volume mapping can be done in docker cli when running an image or in docker-compose.yaml file but not in Dockerfile. Below is an example of docker-compose.yaml file that has volume mapping to each service.
```yaml
services:
  pgdatabase: #name of the service
    image: postgres:18 
    environment:
      POSTGRES_USER: "root"
      POSTGRES_PASSWORD: "root"
      POSTGRES_DB: "ny_taxi"
    volumes: #this map the volume of the service, that is , pgdatabase (the name of the service)
      - "ny_taxi_postgres_data:/var/lib/postgresql" 
      # This maps the `ny_taxi_postgress_data` to the volume inside the container,
      # that is `/var/lib/postgresql`. Why `/var/lib/` ? because in linux they have `var lib` folder structure.
      # In Linux environment, `/var` is the standard directory for *variable data* .
      # We can replace the `ny_taxi_postgress` with other values that corresponds to our real situation.
      # For example, current working directory is `postgress_data`, 
      #just replace the `ny_taxi_progress` with `postgress_data`
    ports:
      - "5432:5432"

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: "admin@admin.com"
      PGADMIN_DEFAULT_PASSWORD: "root"
    volumes:
      - "pgadmin_data:/var/lib/pgadmin"
    ports:
      - "8085:80"

volumes:
  ny_taxi_postgres_data:
  pgadmin_data:
```
As we can see at the above yaml, we have two `volumes`.  
1. At the `services` level, the outer left of the line at the bottom :  
    ```yaml
    volumes:
      ny_taxi_postgres_data:
      pgadmin_data:
    ```
    These 3 lines declare/define that these volumes' name exist and can be used in any services.  We can also configure it further inside this `volumes` .  Read [StackOverFlow](https://stackoverflow.com/questions/72609516/why-do-i-need-to-specify-volumes-twice) for detailed information.  
2. Inside each service
    Self-explanatory

If using CLI like `docker run` , we have to add `-v` flag and then map the volumes, like so :
```
docker run -it --rm \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v ny_taxi_postgres_data:/var/lib/postgresql \ #this is volume mapping or bind mount
  -p 5432:5432 \
  postgres:18
```