# Avency GitHub Workflows

These workflows require separate GitHub environments for each deployment target.
Using environments allows values to be clearly separated or reused through default values.

## Usage of reusable Workflows

If you want to use the reusable workflows, you have to setup some variables and secrets. You can also use them as a template since the workflows just calling custom actions.

### Environments

Create new GitHub environments, for example `testing`, `staging`, and `production`.

Add the following variables (not secrets) to each environment:

- **DOMAIN**: The domain is used exclusively for HTTP checks.
- **DEPLOYMENT_COMPOSE_SOURCE**: The path inside the repository where the deployment files are located (for example `Deployment/Live`).
- **DEPLOYMENT_COMPOSE_TARGET**: The path on the server where files should be copied. :warning: **All files** in the target directory will be overwritten.
- **ENV_FILENAME**: (optional) if there are multiple `.env` files, define which one to use here. A symlink is created during deployment.
- **DEPLOYMENT_COMPOSE_ENVIRONMENT_SOURCE**: (optional) the path of the environment source in the repository. If this is set then `DEPLOYMENT_COMPOSE_SOURCE` should only contain shared files between every environment.

Then create the following secrets:

- **SSH_HOST**: Hostname for the SSH connection.
- **SSH_PORT**: Port used by SSH.
- **SSH_USERNAME**: Username for login.
- **SSH_KEY**: SSH key used for authentication.

### General variables

The following variables can be created under "Secrets and Variables" -> "Variables".
They can also be optionally overridden in each environment.

- **DEPLOY_DO_HTTP_CHECK**: Run an HTTP check for status code `200` before initialization and after deployment (requires `DOMAIN`).
- **DEPLOY_EXEC_CLEAR_CACHE_WARMUP**: Clear and warm up the cache.
- **DEPLOY_EXEC_MIGRATE_DATABASE**: Run database migrations.
- **DEPLOY_EXEC_PUBLISH_RESOURCE**: Publish all static resources.
- **DEPLOY_EXEC_COMMAND_MIGRATION**: Run command migrations.
- **DEPLOY_EXEC_ELASTICSEARCH_INDEX**: Build a classic Elasticsearch index.
- **DEPLOY_EXEC_ELASTICSEARCH_QUEUE**: Create a queue to build the Elasticsearch index (note: the queue is cleared first).
- **DEPLOYMENT_COMPOSE_DO_BACKUP**: Create a backup before deleting the deployment files
- **DEPLOYMENT_COMPOSE_BACKUP_TARGET_FOLDER**: Path where the backup tar file should be placed
- **DEPLOYMENT_COMPOSE_FORCE_RECREATE_CONTAINERS**: Force-Recreate Docker Containers. This should be set to `redis`


If some secrets are identical across environments, they can also be defined globally under "Secrets and Variables" -> "Secrets".

## Workflows

### Build Docker Image

Build and publish a docker image.

The Docker image also always receives the shortened SHA value from the last Git commit as a tag.

```yaml
name: Build new docker image
run-name: Building images for version ${{ github.ref_name }} by @${{ github.actor }}

on:
  push:
    tags: ['*.*.*']
  workflow_dispatch:

jobs:
  php:
    uses: avency/gh-workflows/.github/workflows/build-image.yml@v3
    permissions:
      contents: read
      packages: write
    with:
      docker-registry: ghcr.io # target docker registry i.e. ghcr.io or docker.io
      docker-image-owner: ${{ github.repository }} # the username/owner of the docker registry
      docker-image-name: php # the name of the docker image
      image-is-latest: ${{ github.ref_type == 'tag'}} # mark the image as latest if a tag is created
      build-context: . # docker context path
      build-dockerfile: ./Docker/php-fpm/DockerfileProd # path to the dockerfile
      vulnerability-scan-run: ${{ vars.BUILD_DO_VUL_SCAN_PHP == 'true' }} # should the image be scanned for vulnerabilities
    secrets:
      REGISTRY_USERNAME: ${{ github.actor }} # username for the registry
      REGISTRY_PASSWORD: ${{ secrets.DOCKER_REGISTRY_TOKEN }} # password for the registry
      # a list of all secrets that are needed to build the dockerfile
      BUILD_SECRETS: |
        "composer_token=${{ secrets.COMPOSER_TOKEN }}"
```

### Deploy and initialize Neos Project

```yaml
name: Deploy
run-name: Deploy ${{ github.ref_name }} by @${{ github.actor }}

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Environment to deploy to"
        required: true
        type: choice
        options:
          - testing
          - staging
          - production

concurrency: deploy_to

jobs:
  prepare:
    runs-on: ubuntu-latest
    outputs:
      short_sha: ${{ steps.vars.outputs.short_sha }}
    steps:
      - uses: actions/checkout@v6
      - id: vars
        run: echo "short_sha=sha-$(git rev-parse --short HEAD)" >> "$GITHUB_OUTPUT"

  deploy:
    needs: [prepare]
    uses: avency/gh-workflows/.github/workflows/deploy-neos.yml@v3
    with:
      environment: ${{ inputs.environment }}
      version: ${{ github.ref_type == 'tag' && github.ref_name || needs.prepare.outputs.short_sha }}
      ref-type: ${{ github.ref_type }}
    secrets:
      SSH_HOST: ${{ secrets.SSH_HOST }}
      SSH_PORT: ${{ secrets.SSH_PORT }}
      SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
      SSH_KEY: ${{ secrets.SSH_KEY }}
      BASIC_AUTH_USERNAME: ${{ secrets.BASIC_AUTH_USERNAME }}
      BASIC_AUTH_PASSWORD: ${{ secrets.BASIC_AUTH_PASSWORD }}
```

## Actions

### deploy

Deploy a Compose Project and rebuild the containers. It is possible to merge two folders during the deployment.

[Read more](./.github/actions/deploy/ReadMe.md)

### local-http-check

Check the HTTP Status through the server itself.

[Read more](./.github/actions/local-http-check/ReadMe.md)

### maintenance

Enable/Disable Maintenance

[Read more](./.github/actions/maintenance/ReadMe.md)

### neos-clear-cache

Clear and warm up the Neos Flow Cache 

[Read more](./.github/actions/neos-clear-cache/ReadMe.md)

### neos-migrate-database

Neos Migrate Database

[Read more](./.github/actions/neos-migrate-database/ReadMe.md)

### neos-publish-resources

Publish all static resources for a Neos/Flow instance

[Read more](./.github/actions/neos-publish-resources/ReadMe.md)

### neos-command-migration

Run Command Migrations

[Read more](./.github/actions/neos-command-migration/ReadMe.md)

### neos-elasticsearch-index

Build the elasticsearch index with the default command or with the queue.

[Read more](./.github/actions/neos-elasticsearch-index/ReadMe.md)
