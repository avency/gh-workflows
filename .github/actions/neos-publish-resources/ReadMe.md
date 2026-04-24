# Neos Publish Static Resources

Publish all static resources for a Neos/Flow instance

```yaml
name: Neos Publish Static Resources
run-name: Publish Static Resources for ${{ inputs.environment }}

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Target Environment"
        required: true
        type: environment

jobs:
  init:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: avency/gh-workflows/.github/actions/neos-publish-resources@main
        with:
          compose-service-name: php # optional, the name of the container where the commands should be executed
          compose-service-user: www-data # optional
          deployment-compose-target: ${{ vars.DEPLOYMENT_COMPOSE_TARGET }} # target directoy where the compose.yml is located
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
          SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
          SSH_KEY: ${{ secrets.SSH_KEY }}
```
