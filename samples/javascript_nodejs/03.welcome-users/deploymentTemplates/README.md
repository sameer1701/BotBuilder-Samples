## Deploying via ARM template (with new Resource Group)

Example command:
```bash
az deployment create --name "<name-of-deployment>" --template-file "all-up-template.json" --subscription "<subscription-guid>" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newServerFarmName="<name-of-server-farm>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>"
```

We recommend provisioning Azure resources through ARM templates via the [Azure CLI][ARM-CLI]. It is also possible to deploy ARM templates via the [Azure Portal][ARM-Portal], [PowerShell][ARM-PowerShell] and the [REST API][ARM-REST].

To install the latest version of the Azure CLI visit [this page][Install-CLI].

  [ARM-CLI]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy-cli
  [ARM-Portal]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy-portal
  [ARM-PowerShell]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy
  [ARM-REST]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy-rest
  [Install-CLI]: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest

___

## Bot deployment via Azure CLI
When deploying an ARM template via the Azure CLI, you will perform the following actions:

#### 1. Create an App registration
To create an App registration via the Azure CLI, perform the following command:
```bash
# Replace "displayName" and "AtLeastSixteenCharacters_0" with your specified values.
# The --password argument must be at least 16 characters in length, and have at least 1 lowercase char, 1 uppercase char, 1 special char, and 1 special char (e.g. !?-_+=)
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

This command will output JSON with the key "appId", save the value of this key for the ARM deployment, where it will be used for the `"appId"` parameter. The password provided will be used for the `"appSecret"` parameter.

> *It is also possible to create App registrations via [apps.dev.microsoft.com][Apps-List] or via the [Azure portal][Preview-Portal]. Be sure to also create a password when creating the application.*

  [Apps-List]: https://apps.dev.microsoft.com/#/appList
  [Preview-Portal]: https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade

#### 2. Create a resource group and the Azure resources
```bash
# Pass in the path to the ARM template for the --template-file argument.
az deployment create --name "myDeployment" --template-file "all-up-template.json" --parameters groupName="MyNewResourceGroup" ...
```

#### 3. Retrieve or create necessary IIS/Kudu files via `az bot`
```bash
# For C# bots, it's necessary to provide the path to the .csproj file relative to --code-dir. This can be performed via the --proj-file-path argument
az bot prepare-deploy --code-dir ".." --lang <Csharp or Node>
```

#### 4. Zip up the code directory manually

#### 5. Deploy code to Azure using `az webapp`

```bash
az webapp deployment source config-zip ...
```
___


### Parameters:### Parameters:
```json
"parameters": {
    "groupLocation": {
        "defaultValue": "westus",
        "type": "string",
        "metadata": {
            "description": "Specifies the location of the Resource Group. Defaults to \"westus\"."
        }
    },
    "groupName": {
        "type": "string",
        "metadata": {
            "description": "Specifies the name of the Resource Group."
        }
    },
    "appId": {
        "type": "string",
        "metadata": {
            "description": "Active Directory App ID, set as MicrosoftAppId in the Web App's Application Settings."
        }
    },
    "appSecret": {
        "type": "string",
        "defaultValue": "",
        "metadata": {
            "description": "Active Directory App Password, set as MicrosoftAppPassword in the Web App's Application Settings. Defaults to \"\"."
        }
    },
    "botId": {
        "type": "string",
        "metadata": {
            "description": "The globally unique and immutable bot ID. Also used to configure the displayName of the bot, which is mutable."
        }
    },
    "botSku": {
        "defaultValue": "F0",
        "type": "string",
        "metadata": {
            "description": "The pricing tier of the Bot Service Registration. Acceptable values are F0 and S1."
        }
    },
    "newServerFarmName": {
        "type": "string",
        "metadata": {
            "description": "The name of the App Service Plan."
        }
    },
    "newServerFarmSku": {
        "type": "object",
        "defaultValue": {
            "name": "S1",
            "tier": "Standard",
            "size": "S1",
            "family": "S",
            "capacity": 1
        },
        "metadata": {
            "description": "The SKU of the App Service Plan. Defaults to Standard values."
        }
    },
    "newServerFarmLocation": {
        "type": "string",
        "defaultValue": "westus",
        "metadata": {
            "description": "The location of the App Service Plan. Defaults to \"westus\"."
        }
    },
    "newWebAppName": {
        "type": "string",
        "defaultValue": "",
        "metadata": {
            "description": "The globally unique name of the Web App. Defaults to the value passed in for \"botId\"."
        }
    },
    "alwaysBuildOnDeploy": {
        "type": "bool",
        "defaultValue": "false",
        "metadata": {
            "description": "Configures environment variable SCM_DO_BUILD_DURING_DEPLOYMENT on Web App. When set to true, the Web App will automatically build or install NPM packages when a deployment occurs."
        }
    }
}
```