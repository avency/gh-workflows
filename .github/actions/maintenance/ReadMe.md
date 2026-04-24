# Enable or disable maintenance mode

```yaml
name: Enable/Disable Maintenance Mode
run-name: Enable/Disable Maintenance Mode for ${{ inputs.environment }} by @${{ github.actor }}

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Target Environment"
        required: true
        type: environment
      enable:
        description: Enable or disable maintenance
        type: choice
        options:
          - 'true'
          - 'false'
        

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: avency/gh-workflows/.github/actions/maintenance@main
        with:
          enable: ${{ inputs.enable }}
          container-name: nginx # name of the container where the command should be executed
          deployment-compose-target: ${{ vars.DEPLOYMENT_COMPOSE_TARGET }} # target directoy where the compose.yml is located
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
          SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
          SSH_KEY: ${{ secrets.SSH_KEY }}
```
