# Local HTTP-Check

Check the HTTP Status through the server itself.

```yaml
name: local http check

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
      - name: http check
        uses: avency/gh-workflows/.github/actions/local-http-check@main
        with:
          deployment-compose-target: ${{ vars.DEPLOYMENT_COMPOSE_TARGET }} # target directoy where the compose.yml is located
          domain: ${{ vars.DOMAIN }} # full domain name with https://
          basic-auth-username: ${{ secrets.BASIC_AUTH_USERNAME }}
          basic-auth-password: ${{ secrets.BASIC_AUTH_PASSWORD }}
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
          SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
          SSH_KEY: ${{ secrets.SSH_KEY }}
```
