# container-apps-revision-action

This action creates a new revision of an existing 
[Azure Container App](https://azure.microsoft.com/en-us/services/container-apps/) from a GitHub workflow by providing a
previously built image.

## Prerequisites

* A deployed [Azure Container App](https://azure.microsoft.com/en-us/services/container-apps/)
* A previously published image in either
  * a public container registry such as dockerhub **or** 
  * an azure container registry with MS Entra-ID enabled, and the azure container app having a managed identity with the 
    `AcrPull` RBAC role on the registry
* An app registration in azure whose service principal:
  * is used to authenticate the Github action with Azure
  * has the Contributor Azure RBAC role on the Azure Container App

### `azure/login`

The [`azure/login`](https://github.com/marketplace/actions/azure-login) action is used to authenticate calls using the
Azure CLI, which is used in this action to call the
[`az containerapp up`](https://docs.microsoft.com/en-us/cli/azure/containerapp?view=azure-cli-latest#az-containerapp-up)
command.

## Arguments

Below are the arguments that can be provided to the Azure Container Apps Create Revision GitHub Action.

| Argument              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Required | Default                      |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|------------------------------|
| containerAppName      | The name of the Container App to create a revision for                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | true     |                              |
| envVars               | A multi-line input of key-value pairs to include in the deployment of the new revision. Use the format Key=Value with a new-line to separate each environment variable                                                                                                                                                                                                                                                                                                                                                                                      | false    |                              |
| envVarsSelector       | A comma separated list of environment variables to include in the deployment of the new revision. Use the wildcard character (`*`) to match environment variables that start with a specific string. For example, `Api_*` will match all environment variables that start with `Api_`. Note: selected environment variables will be merged with the provided `envVars` input and take precedence over any environment variables with the same key in the `envVars` list                                                                                     | false    |                              |
| envVarKeyTransform    | Define a transformation to apply to the environment variable keys. Use the format "search=>replace" to replace all instances of search with replace in the environment variable keys. For example, `_=>__` will replace all underscores with double underscores in the environment variable keys                                                                                                                                                                                                                                                            | false    |                              |
| envVarMetadataKeyName | The name of the environment variable that will be used to store the keys of the environment variables that are included in the deployment of the new revision. This list will be used to diff against the environment variables in the next deployment to determine which are obsolete and should be removed from the deployment of the new revision. This diffing process is used to allow for other environment variables to be added to the container app, for example by infrastructure as code, without needing to include them in the deployment here | false    | __DeployMetadata__AppVarKeys |
| healthRequestPath     | The path to use when checking the health of the new revision                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | false    | /health                      |
| imageToDeploy         | The full image name to deploy, including the registry and tag                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | true     |                              |
| logAppRevisionCommand | Log the command used to create the new revision?                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | false    |                              |
| resourceGroup         | The name of the resource group that contains the container app                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | true     |                              |
| testRevision          | If set to true, the action will wait for the new revision to successfully respond to a GET request at the configured health path                                                                                                                                                                                                                                                                                                                                                                                                                            | false    |                              |

## Usage

### Minimal

```yml
steps:

  # Logs in with federated github credential
  # (for more info see: https://docs.microsoft.com/en-us/azure/developer/github/connect-from-azure?tabs=azure-portal%2Clinux#use-the-azure-login-action-with-openid-connect)
  - name: Azure login
    uses: azure/login@v2
    with:
      enable-AzPSSession: true
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

  - name: Create Container App revision
    uses: christianacca/container-apps-revision-action@v1
    with:
      containerAppName: my-test-container-app
      imageToDeploy: mcr.microsoft.com/azuredocs/containerapps-helloworld:latest
      resourceGroup: my-test-container-app-rg
```

This will create a new revision for a Container App named `my-test-container-app` in the resource group named
`my-test-container-app-rg` using the image `mcr.microsoft.com/azuredocs/containerapps-helloworld:latest`.


### Full

```yml

jobs:
  dev:
    runs-on: ubuntu-latest
    environment: dev
    env:
      Api_Key1: 1
      Api_Key2: some value
      ApplicationInsights_ConnectionString: some-connection;string

  steps:
  
    # Logs in with federated github credential
    # (for more info see: https://docs.microsoft.com/en-us/azure/developer/github/connect-from-azure?tabs=azure-portal%2Clinux#use-the-azure-login-action-with-openid-connect)
    - name: Azure login
      uses: azure/login@v2
      with:
        enable-AzPSSession: true
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  
    - name: Create Container App revision
      uses: christianacca/container-apps-revision-action@v1
      with:
        containerAppName: my-test-container-app
        envVars: |
          Other_Key1=Value '1'
          Other_Key2=${{ vars.KEY3 }}
        envVarsSelector: Api_*,ApplicationInsights_*
        envVarKeyTransform: _=>__
        healthRequestPath: /extended-health
        imageToDeploy: clcsoftwaredevops.azurecr.io/web-api-starter/api:0.0.1
        logAppRevisionCommand: true
        resourceGroup: my-test-container-app-rg
        testRevision: true
```

This will create a new revision for a Container App named `my-test-container-app` in the resource group named
`my-test-container-app-rg` using the image `clcsoftwaredevops.azurecr.io/web-api-starter/api:0.0.1`.
The new revision will include environment variables `Other_Key1` and `Other_Key2` along with the environment 
variables that start with `Api_` or `ApplicationInsights_`. 
The keys of the environment variables will be transformed to replace all underscores with double underscores.
The health check will be performed at the path `/extended-health` and the action will wait for the new revision to 
successfully respond to a GET request at that path.
