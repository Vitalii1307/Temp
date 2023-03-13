# Prerequisites

Docker host machine must have the following dependencies installed:
    - docker (v18+)
    - docker-compose (v1.20+)

# Download and configure the environment
    
    cd /opt/airflow/dags_projects

    # init submodule:

        git submodule init

    # You should get:

        # Submodule 'metser' (ssh://git.portaone.com:29418/aux/metser.git) registered for path 'metser'

    # To get all the data of this project run the command:
        
        git submodule update

    # If you get an "Access Denied" error message, you need to configure the .git/config:
        
        nano .git/config 
        
    # Change link

        url = ssh://<YOUR_LOGIN>@git/aux/metser.git

    cd /opt/airflow/docker

    echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=0" > .env


NB: in case you need to deploy apps for development purposes you've to supply "--profile dev"
for every `docker-compose` invocation. To simplify this just add corresponding
environment variable to `.env` file:

    echo "COMPOSE_PROFILES=dev" >> .env

# Build images and initialize AirFlow

    docker-compose build
    docker-compose up airflow-init

# Run

    docker-compose up -d


Make sure that all services/containers are up and running. In particular:

    docker-compose ps



# Further configuration

There is a redpanda service included into the compose file so it is deployed
with the project by default. So one may use a queue service in his DAGs/tasks
however it requires to create a connection in AirFlow (Admin->Connections):

## Connection to Redpanda

    Conn Id: redpanda_default
    Conn Type: HTTP
    Host: redpanda
    Port: 9092
    Extra: {"brokers": ["redpanda:9092"]}


# Attaching other projects to AirFlow instance

The idea is to add every new project to an existing AirFlow instance as a
submodule. All such projects shall be placed in `dag-projects` subfolder:

    dag-projects/<NAME>

The recommended structure for the 3-rd party project is the following:

    /project-directory
    `--/dags             # <-- directory with DAG files
    `--/src              # <-- root dir for source code
        `--/bin          # <-- additional scripts/utils (launched by DAGs)
        `--/templates    # <-- templates (for .sh commands)
        `  `--/sql       # <-- templates for SQL exprs
        `--/etc          # <-- auxiliary data (configs, schemas, whatsoever)
        `--/plugins      # <-- dir for custom plugins
        `--/operators    # <-- dir for custom operators
        `--/hooks        # <-- dir for custom hooks
    `-- .airflowignore   # <-- file with the exclusions

Again, above is the recommended structure (not a strict requirement) however if
one starts a new project and has an option to choose then it's better to stick
with suggested approach.

It is also highly recommended that every separate project has a file with
exclusions `.airflowignore` that excludes all directories but with DAGs. It will
significantly improve the overall performance of AirFlow as it constantly updates
the contents of ${AIRFLOW_HOME}/dags directory and having proper `.airflowignore`
files may help it.

Finally if an attached project requires extra modules for proper functioning one
may require to add them to docker/Dockerfile and rebuild images so that they
contain required libraries and/or dependencies.

# Separation of responsibilities and overriding defaults.

This is based on two principles:

    - developers add required changes to docker/docker-compose.yaml
    - IT Support/devops add customizations to docker/docker-compose.override.yaml
