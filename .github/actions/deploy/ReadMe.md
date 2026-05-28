# Deploy a Compose project

Deploy a Compose Project and rebuild the containers.

## Deploy a whole directory

### Folder Structure

```
.
└── Deployment/
    ├── Production/
    │   ├── compose.yml
    │   └── .env
    ├── Staging/
    │   ├── compose.yml
    │   └── .env
    └── Testing/
        ├── compose.yml
        └── .env
```

### Example Files

```yaml
# Deployment/Production/compose.yml

services:
  php: []
  nginx: []
  # ...
```


### Job Definition

```yaml
name: Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Target Environment"
        required: true
        type: environment

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: avency/gh-workflows/.github/actions/deploy@main
        with:
          version: ${{ github.ref_type == 'tag' && github.ref_name || needs.prepare.outputs.short_sha }}
          ref-type: ${{ github.ref_type }}
          deployment-compose-source: ${{ vars.DEPLOYMENT_COMPOSE_SOURCE }} # => ./Deployment/Production
          deployment-compose-target: ${{ vars.DEPLOYMENT_COMPOSE_TARGET }} # => /docker/deployment/foo-project
          deployment-environment-env-filename: ${{ vars.ENV_FILENAME }} # => .foo-project.env
          # deployment-compose-do-upgrade: true
          # deployment-force-recreate-containers: ${{ vars.DEPLOYMENT_COMPOSE_FORCE_RECREATE_CONTAINERS }}
          # backup-do-backup: ${{ vars.DEPLOYMENT_COMPOSE_DO_BACKUP }}
          # backup-target-folder: ${{ vars.DEPLOYMENT_COMPOSE_BACKUP_TARGET_FOLDER }}
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
          SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
          SSH_KEY: ${{ secrets.SSH_KEY }}
```


## Deploy and merge two directories

If you have a base directory with all services and a second directory with every environment, use this job definition instead.

### Folder Structure

```
.
└── Deployment/
    ├── Base/
    │   └── lib/
    │       ├── compose.base.yml
    │       ├── compose.cronjob.a.yml
    │       └── compose.cronjob.b.yml
    ├── Production/
    │   ├── compose.yml
    │   └── .env
    ├── Staging/
    │   ├── compose.yml
    │   └── .env
    └── Testing/
        ├── lib/
        │   └── compose.cronjob.a.yml
        ├── compose.yml
        └── .env
```

The file `Deployment/Testing/lib/compose.cronjob.a.yml` will override the file from `./Deployment/Base/lib/compose.cronjob.a.yml` during the deployment.


### Example Files

```yaml
# Deployment/Base/lib/compose.base.yml

services:
  php: []
  nginx: []
  # ...
```

```yaml
# Deployment/Production/compose.yml

includes:
  - ./lib/compose.base.yml
```

### Job Definition

```yaml
name: Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Target Environment"
        required: true
        type: environment

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: avency/gh-workflows/.github/actions/deploy@main
        with:
          version: ${{ github.ref_type == 'tag' && github.ref_name || needs.prepare.outputs.short_sha }}
          ref-type: ${{ github.ref_type }}
          deployment-compose-source: ${{ vars.DEPLOYMENT_COMPOSE_SOURCE }} # => ./Deployment/Base
          deployment-compose-target: ${{ vars.DEPLOYMENT_COMPOSE_TARGET }} # => /docker/deployment/foo-project
          deployment-environment-source: ${{ vars.DEPLOYMENT_COMPOSE_ENVIRONMENT_SOURCE }} # => ./Deployment/Production
          deployment-environment-env-filename: '' # not needed, put the env file as .env into the environment folder
          deployment-compose-do-upgrade: true
          # deployment-force-recreate-containers: ${{ vars.DEPLOYMENT_COMPOSE_FORCE_RECREATE_CONTAINERS }}
          # backup-do-backup: ${{ vars.DEPLOYMENT_COMPOSE_DO_BACKUP }}
          # backup-target-folder: ${{ vars.DEPLOYMENT_COMPOSE_BACKUP_TARGET_FOLDER }}
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
          SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
          SSH_KEY: ${{ secrets.SSH_KEY }}
```
