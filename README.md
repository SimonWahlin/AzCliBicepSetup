# AzCliBicepSetup
Simple composite action that will install and cache bicep for az cli

Will install latest bicep version on first successful run and then cache that version for future runs.  
This is intended as a workaround for problems with rate limiting on the GitHub API since we no longer  
will be installing bicep on each run.  

> Note:
> Does not work with the Azure/CLI actions since that action runs in a separate container.  
> When using public runners, az CLI is already installed so really no need to run Azure/CLI,  
> just use a simple bash or pwsh step unless you require a specific version of the cli.  

Usage Example:

```
on:
  workflow_dispatch:

jobs:
  deployBicep:
    runs-on: ubuntu-latest
    name: deploy a bicep file
    steps:
      - uses: actions/checkout@v3
      
      # This step shows that bicep is not installed on start of pipeline
      - shell: bash
        run: az bicep version
        continue-on-error: true
      
      # This action will restore bicep from cache or install if not found in cache
      - uses: SimonWahlin/AzCliBicepSetup@v1
      
      # This step shows that bicep is now installed
      - shell: bash
        run: az bicep version
        
      # This action will authenticate to Azure (requires CREDENTIALS secret)
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.CREDENTIALS }}

      # Simple bash step that lists current bicep version
      # and deploys a template (az cli is included in ubuntu runner)
      - name: "Deploy template"
        shell: bash
        run: |
          az bicep version
          az deployment sub create --location westeurope --template-file main.bicep
```
