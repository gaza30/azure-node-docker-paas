# Azure Resource Manager Template

## Overview

Use supplied [azuredeploy.json](azuredeploy.json) template to deploy the resources we need into Azure. An easy way to do this is via the Azure portal, simply click this button:  
  
[![deploy](azuredeploy.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fbenc-uk%2Fazure-node-docker-paas%2Fmaster%2Fazure%2Fazuredeploy.json)  
(Tip. Right click and open in new tab)

## Resources Deployed
* Azure Container Registry
* Linux App Service Plan
* Linux Web App
* Storage Account

## Parameters
The template will prompt you for several parameters:
* `siteName` : The name of your web app, **Note: This must be globally unique as it is registered in DNS** 
* `registryName` : The name of your docker registry, **Note: This must be globally unique as it is registered in DNS**  
* `dockerImageName` : The name of your application Docker image 
* `dockerImagePort` : Port your application listens on