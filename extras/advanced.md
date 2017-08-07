# Alternative Flow for Demos 

Alternative flow is to create the Linux web app using an infrastructure as code approach in the VSTS release step, via an ARM template. This is slightly more "real" CD than manually creating some static resources to deploy into.  
In this version of the flow, skip step 4 completely, but instead create an ACR instance using the Azure CLI, or you can use the portal if you wish. 

### Create Azure Container Registry using CLI

Create an ACR instance using Azure CLI v2. [Install instructions](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)  
```
az group create -l westus -n MyResGroup 
az acr create --admin-enabled --sku Basic --verbose -l westus -n demoregfoo123 -g MyResGroup 
```

### VSTS release defintion - Infrastructure as Code 

Rather than use static pre-deployed resources, we'll deploy the Azure resources at the release stage across multiple environments. Using the following [ARM template](https://github.com/benc-uk/azure-arm/tree/master/paas-web/linux-acr-existing) which is hosted in a separate Github repo.
The release definition is likely too laborious to create manually, esp. in a demo scenario so import the definition from the JSON file found in the [extras folder](extras/)  

Once imported:
- Modify the 'Variables' section and enter your ACR information; registry name, username & password. 
  - Make sure the sitename variable will be a globally unique name, the default includes a prefix of environment name e.g. Dev/Test/Staging so that each environment has a unique name and URL
- In each environment - Click on **'Run on agent'** and select *"Hosted"*
- Select an Azure subscription in the dropdown for any of the Azure tasks, clicking **'Authorize'** if necessary. You'll need to set this in both tasks across the 3 environments (so six times)
- For CD to work, click **'Artifacts'** and chose your build as a source, then go into **'Triggers'** and enable **'Continuous Deployment'**


