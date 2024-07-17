# Avency GitHub Workflows

## InitNeos

Execute Migrations and Commands after a deployment.

### Implementation

#### With Container-Name

```yaml
# .github/workflows/deploy-to-live.yml

jobs:
  # [...]

  init:
    uses: avency/gh-workflows/.github/workflows/InitNeos.yml@main
    concurrency: deploy-to-live
    with:
      containername: ${{ secrets.CONTAINERNAME }}
      url: https://${{ secrets.PROJECT_DOMAIN_LIVE }}/
      exec_clear_cache_warmup: true
      exec_migrate_database: true
      exec_publish_resource: true
      exec_command_migration: false
      exec_elasticsearch_index: false
      exec_elasticsearch_queue: false
    secrets:
      host: ${{ secrets.HOST }}
      username: ${{ secrets.USERNAME }}
      key: ${{ secrets.AVENCY_SSH_KEY }}
```

##### Special Inputs

###### containername

The name of the appropriate container in which the commands should be executed.

#### InitNeos (Docker Composer)

```yaml
# .github/workflows/deploy-to-live.yml

jobs:
  # [...]

  init:
    uses: avency/gh-workflows/.github/workflows/InitNeosCompose.yml@main
    concurrency: deploy-to-live
    with:
      compose_folder: ${{ inputs.deployment-compose-target }}
      compose_service_name: php
      compose_service_user: www-data
      url: https://${{ secrets.PROJECT_DOMAIN_LIVE }}/
      exec_clear_cache_warmup: true
      exec_migrate_database: true
      exec_publish_resource: true
      exec_command_migration: false
      exec_elasticsearch_index: false
      exec_elasticsearch_queue: false
    secrets:
      HOST: ${{ secrets.HOST }}
      SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
      SSH_KEY: ${{ secrets.SSH_KEY }}
```

##### Special Inputs

###### compose_folder

The folder where the docker compose file is located.

###### compose_service_name

The name of the container (service) in which the commands should be executed.

###### compose_service_user

The name of the user who should execute the commands. (default: www-data)


##### Inputs

### Inputs

The inputs are defined under `jobs.init.with`.

#### url

The url with which the project can be accessed

#### exec_clear_cache_warmup

The cache will be flushed and warmed up.

#### exec_migrate_database

Executes all doctrine migrations.

#### exec_publish_resource

Publishes the static resources.

#### exec_command_migration

Executes pending command migrations. [Requires additional Package](https://github.com/swisscomeventandmedia/Swisscom.CommandMigration)

#### exec_elasticsearch_index

Builds an ElasticSearch Index. Can not be used with `exec_elasticsearch_queue`. [Requires additional Package](https://github.com/Flowpack/Flowpack.ElasticSearch.ContentRepositoryAdaptor)

#### exec_elasticsearch_queue

Builds an Elasticsearch Queue Index. Can not be used with `exec_elasticsearch_index`. [Requires additional Package](https://github.com/Flowpack/Flowpack.ElasticSearch.ContentRepositoryQueueIndexer)

### Secrets

#### HOST:

The SSH Host name

#### SSH_USERNAME

Username for ssh.

#### SSH_KEY

SSH Key

#### BASIC_AUTH_USERNAME

If the host is protected with basic auth then you need to set the username.

#### BASIC_AUTH_PASSWORD

The password for the basic auth
