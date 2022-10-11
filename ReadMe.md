# Avency GitHub Workflows

## InitNeos

Execute Migrations and Commands after a deployment.

### Implementierung

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
    secrets:
      host: ${{ secrets.HOST }}
      username: ${{ secrets.USERNAME }}
      key: ${{ secrets.AVENCY_SSH_KEY }}
```

### Inputs

The inputs are defined under `jobs.init.with`.

#### containername

The name of the appropriate container in which the commands should be executed.

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