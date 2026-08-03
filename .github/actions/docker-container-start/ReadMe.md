# Docker Container Start

Start docker containers (optional limited to a profile).


## Job Definition

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
      - uses: avency/gh-workflows/.github/actions/docker-container-start@main
        with:
          compose-source: ${{ vars.DEPLOYMENT_COMPOSE_SOURCE }} # => ./Deployment/Production
          compose-profile-name: ${{ vars.DEPLOYMENT_COMPOSE_INIT_PROFILE }}
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
          SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
          SSH_KEY: ${{ secrets.SSH_KEY }}
```

## Example Files

```yaml
# compose.yml

servies:
  php:
    profiles:
      - base
      - production
  nginx:
    profiles:
      - base
      - production
  
  foobar-worker:
    profiles:
      - production
```

```text
# .env

COMPOSE_PROFILES=production
```
