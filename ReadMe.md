# Avency GitHub Workflows

These workflows require separate GitHub environments for each deployment target.
Using environments allows values to be clearly separated or reused through default values.

## Usage

### Environments

Create new GitHub environments, for example `testing`, `staging`, and `production`.

Add the following variables (not secrets) to each environment:

- **DOMAIN**: The domain is used exclusively for HTTP checks.
- **DEPLOYMENT_COMPOSE_SOURCE**: The path inside the repository where the deployment files are located (for example `Deployment/Live`).
- **DEPLOYMENT_COMPOSE_TARGET**: The path on the server where files should be copied. :warning: **All files** in the target directory will be overwritten.
- **ENV_FILENAME**: (optional) if there are multiple `.env` files, define which one to use here. A symlink is created during deployment.

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
    tags: ['*.*.*'] # build image after a tag
  workflow_dispatch: # allow manual build of a docker image

jobs:
  php:
    # x-release-please-start-version
    uses: avency/gh-workflows/.github/workflows/build-image.yml@1.4.0
    # x-release-please-end
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
name: Deploy to Testing
run-name: Deploy ${{ github.ref_name }} to Testing by @${{ github.actor }}

on:
  workflow_dispatch:

concurrency: deploy_to_testing

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
    # x-release-please-start-version
    uses: avency/gh-workflows/.github/workflows/deploy-neos.yml@1.4.0
    # x-release-please-end
    with:
      environment: 'testing' # replace with the name of the environment
      version: ${{ github.ref_type == 'tag' && github.ref_name || needs.prepare.outputs.short_sha }}
      ref-type: ${{ github.ref_type }}
    secrets:
      SSH_HOST: ${{ secrets.SSH_HOST }}
      SSH_PORT: ${{ secrets.SSH_PORT }}
      SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
      SSH_KEY: ${{ secrets.SSH_KEY }}
      SSH_PORT: ${{ secrets.SSH_PORT }}
```
