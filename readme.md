# Azure DevOps Lab with Node.js, Azure, Docker & VSTS

## Synopsis
#### From first principals and nothing but VS Code, an empty directory and a command terminal we will create a working web Node.js app running in Azure web platform services & Docker, deployed via an automated DevOps CI/CD pipeline

The scenario will cover:
* Node.js
* Docker
* Azure Container Registry 
* Azure Web App (Linux)
* Visual Studio Team Services (VSTS)

You do not need to be an Node.js or JavaScript expert for the coding part but you will need to make basic changes to a JS file. Likewise no prior experience with VSTS and Azure is required (but obviously beneficial). We will also spend some time with Docker registries and image building & tagging. You will be able to complete the lab with either a Windows or Mac machine, but only Windows has been tested.

The high level overall flow is:
* Generate Node.js Express app
* Add Docker support
* Git repo setup
* Deploy resources in Azure from template
* Creation of VSTS project and code repo
* Push of git repo into VSTS
* Create & run VSTS build definition - generate Docker Image
* Create & run VSTS release definition - deploy Docker container
* Resulting in our Node app deployed & running in a Azure Linux Web App

---

## Pre-requisites 
:warning: **Do not ignore this part!** :warning:  
You will need the following things set up and installed on your machine: 
* An active [Azure subscription](https://portal.azure.com/). If you do not have a subscription:
  * You may have been given an [Azure Pass](https://www.microsoftazurepass.com/) card & code, please follow the steps given to activate your new subscription.
  * OR - create a [free Azure account and subscription](https://azure.microsoft.com/en-gb/free/)
* An active [VSTS Account](https://app.vsaex.visualstudio.com/)
  * If you don't have an account, [create a free VSTS account](https://www.visualstudio.com/en-gb/docs/setup-admin/team-services/sign-up-for-visual-studio-team-services)
* Install [Node.js](https://nodejs.org/en/download/)
* Install [VS Code](https://code.visualstudio.com/download)
  * Required extension: Docker (Ctrl+P `ext install vscode-docker`)
* Install git; [Git for Windows](https://git-scm.com/download/win) or [Git for Mac](https://git-scm.com/download/mac)
* Optional but strongly recommended: [Git credential manager](https://www.visualstudio.com/en-us/docs/git/set-up-credential-managers)

## Initial Setup Steps
Overview of steps:
* Create a new VSTS account, or new project if you already have an account
* If you've never run git before, run these commands (modifying with your details as required):
 ```
git config --global user.email "your-email@example.com"
git config --global user.name "Your Name"
git config --global credential.helper manager
```

---

# Main Exercise Flow
With all the setup pre-reqs complete, what follows is the full step by step guide to the exercise 

## 1. Create Node.js / Express web app
We'll use the well known [Express](https://expressjs.com/) web framework for Node.js, and its builtin generator to create a skeleton application.   
Open a command prompt or terminal and run the following commands:
```bash
npm install express-generator -g
express --view=pug --git myapp
cd myapp
npm install
```
This will use NPM (Node Package Manager) to install the Express generator, then create a new app and folder called 'myapp', then install the Node modules required.
> Note. You can call the app folder anything you like, you don't need to call it 'myapp'

Now open your project folder in VS Code
```bash
code .
```
Take a look around, there's not a huge amount to see, but we'll probably dive into the `views` folder in a second to customise the app a little.

From VS Code, press `Ctrl+'` to open the builtin terminal and run:
```bash
npm start
```
This will start the Node/Express app + webserver which will be listening on port 3000, so open [`http://localhost:3000`](http://localhost:3000) in your browser to see your app.  

Looks pretty dull eh? It's not the most exciting starting page for sure. If you go back to VS Code and open ***views/index.pug*** you can change what your app shows. This file is a view layout in a format called 'Pug', which has a special syntax like very minimal HTML, you can find out the basics [here](https://www.sitepoint.com/jade-tutorial-for-beginners/). This is an example:
```pug
extends layout

block content
  h1 Linux PaaS - Demo Web App
  h3 Microsoft &hearts; Open source
  h3 Let's run this in Azure with Docker
  button OK Great!
```
Any changes to Pug views are picked up without needing to restart Node, so just refresh the browser.  
How much you want to personalize and tweak it from here is up to you. If you know CSS then some quick edits to ***public/stylesheets/style.css*** can make things much less of an eyesore.  

Once you're happy to move on to the next step, press `Ctrl+C` in your VS Code terminal to stop Node from running.


## 2. Containerize the app 
Now we'll add Docker support to the app to containerize it. The Docker extension for VS Code makes this super easy:   
* Press `Ctrl+Shift+P` then type *"docker"*
* Select **'Add docker files to workspace'**
* Choose **'Node.js'** for application platform
* Choose **'3000'** for the port (no quotes)

This will add three files to your project, the two compose YAML files we can ignore. The **Dockerfile** is what we're interested in. We don't need to make any changes but if you're unfamiliar with Docker, it's worth opening and looking at, if you've created Docker images before, then move on.  
Here is some high level explanation of what the **Dockerfile** is doing:  
- Since, a Docker image is nothing but a series of layers built on top of each other, you always start from a base image. The `FROM` command sets this base image, here we're using an image pre-built with Node.js. This is an official image published by the Node foundation and [hosted on Dockerhub](https://hub.docker.com/_/node/). The default image is the Alpine Linux variant, which means it is very small and lightweight  
- The series of `COPY` and `RUN` commands go about running the `npm install` just as we did earlier and also copying in your app source into the image.  
- The `EXPOSE` command is a hint which ports your application and will be listening on, and need to be mapped out to the container when it runs.  
- The last `CMD` part is what starts the app up, just as we did with `npm start`.  


## 3. Create git repo and push to VSTS
To start to get a CI/CD pipeline going we need our code in source control so let's create a git repo and add our code to it. You can do this from the source control view in VS Code; the branch icon on the lefthand toolbar (also `Ctrl+Shift+G`) or from the integrated terminal you can run:
```
git init
git add .
git commit -m "First commit"
```
To get the code up into VSTS, go into your new project and we'll need to set the remote origin for the repo and push it.  
> Note. You will get the correct URL & syntax for this part by expanding the *"push an existing repository from command line"* section of the project start page or from the 'code' section in VSTS
```
git remote add origin https://<vsts-account>.visualstudio.com/_git/<project>
git push -u origin --all
```
If you have the git credential manager installed, authentication should should automatically pop up, so login with your VSTS account details.  
If you have trouble and you get username/password prompt at the command line, you have the option of manually creating git credentials by going into *VSTS --> Code --> Generate Git credentials*

You can validate this has worked by looking to the 'Code' section on the main VSTS menu.


## 4. Deploy resources to Azure
Switching from VSTS to Azure for a moment, we'll set about creating the cloud resources we need to support our application and release pipeline. We'll do this by deploying a template, called an ARM (Azure Resource Manager) template. This template is a JSON file that describes all the resources to be built, and any input parameters we want to supply. This template has been created for you and will deploy into Azure:
* Azure Container Registry
* Linux App Service Plan
* Linux Web App
* Storage Account

There are many ways to deploy this template, but we'll use a simple "Deploy to Azure" button. Go into the [Azure folder under this repository](azure/) and you should see a page with big blue button which will jump into the Azure portal ready to deploy our template. You will be prompted for a number of things: 
* *Resource group* - This group will hold all your resources. Choose any name you like
* *Location* - Pick a local Azure region e.g. North or West Europe or one of the UK regions
* *Site Name* - This is the name of your new site and Linux web app
  :warning: Important! **Pick a globally unique name** as this site will be registered in global DNS. Suggestion, to use your initials and/or date, e.g. *`mysitebenc-1204`*
* *Registry Name* - This is the name of your new Docker registry, same as above *must be a globally unique name!* and **only use letters and numbers**
* *Docker Image Name* - Leave this as `myapp` for now
* *Docker Image Port* - Leave this as `3000` for now

Tick the **'I agree to the terms and conditions stated above'** checkbox. If you've never used the Azure portal before, I also advise ticking the **'Pin to dashboard'** option too. Then click **'Purchase'** to start deployment.  
Deployment should take about 1-2 minutes...  

Once deployed, go into the new resource group (if you ticked the option to pin to your dashboard you will be taken there automatically). We don't need to do anything here, but it's good to validate the deployment & spend 5 minutes looking at what as been deployed, especially if you haven't used Azure before.  

<details>
  <summary>View screenshots of the Azure resources (Click to expand/collapse)</summary>

  ![build-1](imgs/res-group.png)
</details> 

## 5. VSTS build process
Back in VSTS and in your new project, click into *Build & Release --> Build* from the main menu. If you are prompted with a blue bar at the top to try the new build editor, definitely enable it by clicking **'Try It'**, otherwise enable it from hovering over your account top right and going into **'Preview features'**. 

* Click **'+ New Definition'**
* Choose **'Container (PREVIEW)'** as the starting template (near the bottom)
* Give a sensible name to the definition, e.g. *"Build MyApp Docker Image"*
* Click on **'Process'** above the list of tasks, and change the **'Default agent queue'** to **'Hosted Linux Preview'**
* Click the **'Build an image'** task in the list, change the following:
  * Pick your Azure subscription from the drop down, then click **'Authorize'**, this will take about 30 seconds
  * Now click the **'Azure Container Registry'** dropdown and select your registry you just built
  * Tick these options: **'Qualify Image Name'** & **'Include Latest Tag'**
  * Change the **'Image Name'** to `myapp:$(Build.BuildId)`
* Click the **'Push an image'** task in the list, change the following:
  * Pick your Azure subscription again from the drop down
  * Now click the **'Azure Container Registry'** dropdown and select your registry you just built
  * Tick these options: **'Qualify Image Name'** & **'Include Latest Tag'**
  * Change the **'Image Name'** to `myapp:$(Build.BuildId)`
* Click **'Add Task'**, search for *"pub"* and add the task called **'Publish Build Artifacts'**
* Click the new task:
  * Change the **'Contents'** to two asterisks `**`
  * Change **'Artifact Name'** to anything you like e.g. `stuff`
  * For the **'Artifact Type'** pick **'Server'**
* Final step:
  * Click on **'Triggers'** at the top of the definition and enable **'Continuous Integration'**

<details>
  <summary>View screenshots of this step (Click to expand/collapse)</summary>

  Screenshots of the four major steps are: 
  ![build-1](imgs/build-1.png)
  ![build-2](imgs/build-2.png)
  ![build-3](imgs/build-3.png)
  ![build-3](imgs/build-4.png)  
  ![build-4](imgs/build-5.png)  
</details>  

Phew! That's a lot of manual steps, sorry!  
A working build definition is [provided here as a JSON file](vsts-definitions/build-myapp-docker-image.json), which you can import in. This will save you a little work, However you will still need to modify the tasks to use your Azure subscription and your new Azure Container Registry

Click on **'Save & queue'** then queue a new manual build in order to validate everything. You can sit an watch it run which should take less than 3 mins, or jump to the next step and come back here to check and fix any problems


## 6. VSTS release process
Our release process is slightly unorthodox. For the sake of simplicity with this exercise, we've pre-deployed our resources in the step above. The way Azure Linux Web Apps work is they deploy a Docker container for you to spin up your app. As a PaaS service this is done seamlessly for you behind the scenes. To get the Web App to redeploy the container with your new build (or updated build) we just need to restart the Web App. So our release process is simply restarting the Web App in Azure.

Note. These steps are taken with the new style VSTS release editor, which you may be prompted to turn on (click 'Try It' if so), otherwise it is enabled in the "Preview features" section of your profile.

On main menu click into *Build & Release --> Releases*:
* Click **'+ New Definition'**
* Pick **'Start with an empty definition'** or the **'Empty process'** as your starting template
* Name your environment something sensible like *"Dev"* or *"Test"*
* Click the pencil at the top and rename the definition something sensible e.g. *"Release to Azure"*
* Click in the **'Add artifact'** circle
  * Pick your project & new build definition as the source, and click **'Add'**
* Click **'Tasks'** on the menu
  * Under the "Agent phase" click the plus, find the **'Azure App Service Manage (PREVIEW)'** task, click **'Add'**
* Change the following options on the task:
  * Azure subscription: pick from the drop down
  * Action: **"Restart App Service"**
  * App Service name: select your site name you deployed earlier from the dropdown
* Click **'Save'** button at the top
* Click **'+ Release'** then **'Create Release'**
* Take the defaults and click **'Queue'**
* Where it says **_"Release Release-1 has been created."_** click into _Release-1_ and click on **'Logs'**
* Validate that the site has been restarted

<details>
  <summary>View screenshots of the release definition (Click to expand/collapse)</summary>

  ![rel-1](imgs/vsts-rel1.png)
  ![rel-2](imgs/vsts-rel2.png)
</details> 

A working release definition is [provided here as a JSON file](vsts-definitions/release-to-azure.json), which you can import in. This will save you a little work, However you will still need to modify the tasks to use your Azure subscription and your correct Web App site name


## 7. View deployed site & trigger pipeline
OK the part we've been waiting for... assuming your build and release are working, the Node.js app should now be deployed to your site in Azure. The URL of your website is `http://<sitename>.azurewebsites.net` you can also go to the site via the Azure portal; click into the App Service (your web app) and in overview section is a clickable URL.  

> Note. Be patent! As the very first time the site loads Azure will start & deploy the underlying Docker container, which can take a minute or two. So be prepared to wait & refresh once or twice. If you get a permanent 503 error then something has gone wrong and Azure can't start your container.  

To test the end to end automated CI/CD process, go back to you app in VS Code, make a small change to ***views/index.pug*** just put in some extra text or whatever you like. Then in the VS Code source code view (3rd icon from top on the far right) commit your change with a comment, then click the '…' and pick 'Push' to push this change into VSTS.  
This in turn should trigger the build to run (CI), then when that completes it will trigger the release (CD) and update the site with your changed code.  

Job done!  

---

# Summary
You should now have a containerized Node.js application, hosted & running in Azure in a Docker/Linux based PaaS web app. Which is supported with a private docker registry, and fully automated build & release pipeline. We've covered a lot of ground, but hopefully this exercise took less than 2 hours and was educational.  

There's obviously lots more that can be done to enhance and extend this scenario, some ideas to try out:
* Add automated testing to the release, a simple one is just checking URL is available via this [marketplace task](https://marketplace.visualstudio.com/items?itemName=saeidbabaei.checkUrl&showReviewDialog=true)
* Deploy your Azure resources via templates (infrastructure as code) see the [advanced notes](extras/advanced.md) for some guidance on this. 
* Create multiple environments in your release (dev/test/staging/pre-prod etc.)
* Write a better app ;)

---

# Appendix

## Suggested cleanup & removal tasks
 * If you are using the free Azure credit or an Azure pass, it is suggested you remove your web app or scale it down to a B1 (Basic-1) level to save costs and make the most of the allowance.
 * Deleting the resource group created earlier will remove all the resources from Azure.
 * From the admin page in VSTS `https://<your-account>.visualstudio.com/_admin` you remove the project if you wish.
