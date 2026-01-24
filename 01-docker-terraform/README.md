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

# Dockerfile  
## What is Dockerfile?
Put simply, Dockerfile (without extension file) is a recipe card or assembly instructions card. It is a list of steps that tells a robot (in this case, the Docker Engine) how to build fully furnished room (the container) from scratch, like so:  
1. Start with a blank room --> this is the `base image`  
2. Bring in the furniture --> COPY the code  
3. Install appliances, lights, etc --> run the commands  
4. Plug everything --> entrypoint

## Example of a Dockerfile from DataTalksClub
A Simple Dockerfile with uv
```dockerfile
# Start with slim Python 3.13 image
FROM python:3.13.10-slim

# Copy uv binary from official uv image (multi-stage build pattern)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

# Set working directory
WORKDIR /app

# Add virtual environment to PATH so we can use installed packages
ENV PATH="/app/.venv/bin:$PATH"

# Copy dependency files first (better layer caching)
COPY "pyproject.toml" "uv.lock" ".python-version" ./
# Install dependencies from lock file (ensures reproducible builds)
RUN uv sync --locked

# Copy application code
COPY pipeline.py pipeline.py

# Set entry point
ENTRYPOINT ["uv", "run", "python", "pipeline.py"]
```

# Terraform

## What is Terraform?
- A recipe or a blueprint that will be applied to our servers or cloud environment.  
## Why do we need Terraform?
- Automate reproducibility environment. Deploying in cloud by using IaC (infrastructure as a code) like Terraform can help to automate, manage, scale the cloud infrastructure without clicking in the cloud console here and there. This manual clicking is prone to error. By using declarative code, engineer can provision, track, version, and destroy the infrastructure efficiently while ensuring consistency across cloud environments.  
- We can manage multi-cloud using IaC
- Terraform has `terraform plan` command that allows us to preview the changes Terraform will make before we apply them [Terraform Plan Documentation](https://developer.hashicorp.com/terraform/cli/commands/plan)  
- Terraform has `terraform destroy` that is used to permanently remove all the infrastructure resources previously provisioned by our config and state file. [Terraform destroy Documentation](https://developer.hashicorp.com/terraform/cli/commands/destroy). Very convenient to have destroy command, because we can spin up our cloud env for a few minutes, do some testing and then deprovision the cloud resources. It saves cost.


## Things to remember
1. Storing credentials (keypair/service account key for accounts or roles provided by the cloud) must be done right.  
    :x: Never hard code the credentials.  
    :white_check_mark: Export credentials into a variable using `export` cli  
2. Always include [Terraform gitignore](https://github.com/github/gitignore/blob/main/Terraform.gitignore) in .gitignore and include any `*.json`

## How to use Terraform?
Read and copy paste from [documentation of Terraform providers](https://registry.terraform.io/browse/providers). Adjust value based on our real condition.
