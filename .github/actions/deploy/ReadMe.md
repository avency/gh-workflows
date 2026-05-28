# Deploy a Compose project

Deploy a Compose Project and rebuild the containers.

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
          deployment-compose-source: ${{ vars.DEPLOYMENT_COMPOSE_SOURCE }}
          deployment-compose-target: ${{ vars.DEPLOYMENT_COMPOSE_TARGET }}
          env-filename: ${{ vars.ENV_FILENAME }}
          # deployment-compose-do-upgrade: true
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
          SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
          SSH_KEY: ${{ secrets.SSH_KEY }}
```
