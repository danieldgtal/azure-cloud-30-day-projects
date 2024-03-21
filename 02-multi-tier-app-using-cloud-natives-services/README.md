# 02 - Architect and deploy a multi-tier application using cloud-native services (e.g., web server, database,
caching layer).
This guide is a part of the CozyCloud Project Challenge, where I took ownership of implementing the project idea from conception to completion. While the initial idea didn't originate from me, I meticulously executed the project for practical purposes, and this piece serves as my process documentation.
## Introduction
<p> I deployed a secure N-tier web application using cloud-native services such as Azure App Service, DNS, and others on     Azure, and integrated a caching layer. This tutorial is adapted from Microsoft's documentation on building an N-tier     application using Azure App Service <a href="https://learn.microsoft.com/en-us/azure/app-service/tutorial->Secure N-     tier Web App.</a>
</p>
## Prerequisite
. Active Azure Sub \
. Github account \
. VSCode 
## Azure Services used
* Azure App Service P1V3
* Azure Service Principal
* Azure Private DNS etc.
                                                                                                                                                                                                                                                                                            
## Deployment Process
1. Create two instances of a web app
2. Create network infrastructure
3. Configure virtual newtwork integration in your frontend web app
4. Enable deployment to back-end web app from internet
5. Lock down FTP and SCM access
6. Configure continuous deployment using GitHub Actions
7. Use a service principal for GitHub Actions deployment
8. Validate connections and app access



### Scenario architecture
<p>Many applications consist of multiple components, such as a frontend facing the public, a backend restricted from the public, or storage restricted from the public. The architecture for this project includes:</p>
                                                                                                                      <ul>
  <li><strong>Virtual network</strong>  Contains two subnets, one is integrated with the front-end web app, and the other has a private endpoint for the back-end web app. The virtual network blocks all inbound network traffic, except for the front-end app that's integrated with it.</li>
  <li><strong>Front-end web app</strong>  Integrated into the virtual network and accessible from the public internet.      </li>
  <li><strong>Back-end web app</strong>  Accessible only through the private endpoint in the virtual network.</li>
  <li><strong>Private endpoint</strong>  Integrates with the back-end web app and makes the web app accessible with a       private IP address.</li>
  <li><strong>Private DNS zone</strong>  Lets you resolve a DNS name to the private endpoint's IP address.</li>
  <li>Public traffic to the back-end app is blocked</li>
    <li>Outbound traffic from App Service is routed to the virtual network and can reach the backend app</li>
  <li>App Service is able to perform DNS resolution to the back-end app.</li>
</ul>
                                                                                                                                                         
<p>The following diagram shows the architecture you'll create during this project</p>

![PJ02-SS01](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/10e33597-92b8-415b-ad61-6550942ef31f)


### Steps 01 - 05 Create two instances of a web app
Follow the Microsoft documentation from Step 1 to Step 5 (Instances creation to Lock down FTP and SCM access
<a href="https://learn.microsoft.com/en-us/azure/app-service/tutorial-secure-ntier-app">Ms Docs Link</a>

### Step 06 Configure Continuous Depeloyment Using GitHub Actions
Fork this two repo from github. The frontend and the Backend nodejs app \
<a href="https://github.com/seligj95/nodejs-backend">Backend Repo</a> \
<a href="https://github.com/seligj95/nodejs-frontend">Frontend Repo</a>

![Screenshot 2024-03-21 140902](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/8de82020-c476-4cdc-a7fd-3b42825e39e5)

<code>az ad sp create-for-rbac --name "myApp" --role contributor --scopes /subscriptions/<subscription-id>/resourceGroups/$groupName/providers/Microsoft.Web/sites/<frontend-app-name> /subscriptions/<subscription-id>/resourceGroups/$groupName/providers/Microsoft.Web/sites/<backend-app-name> --sdk-auth</code>

<p>The output is a JSON object with the role assignmet credentials that provide access to your App Service apps. Copy This JSON object for the next step.</p>

<p>7.To store the service principal's credentials as GitHub secrets, go to one of the forked sample repositories in GitHub and go to <strong>Settings</strong> &gt; <strong>Security</strong> &gt; <strong>Secrets and variables</strong> &gt; <strong>Actions</strong>.</p>

<p>8. Select <strong>New repository secret</strong> and create a secret for each of the following values. The values can be found in the json output you copied earlier.</p>

![PJ02-SS03](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/13af97eb-6469-4434-ba52-cc15f692b1bf)

<p>9. Repeat this process for the other forked sample repository.</p>
<p>10. To set up continous deployment with GitHub Actions, sign in to the Azure <a href="portal.azure.com">Portal</a></p>
<p>11. Navigate to the Overview page for your front-end webapp </p>
<p>12. In the left pane, select Deployment Center. Then select Settings.</p>
<p>13. In the source box, select "GitHub from the CI/CD options.</p>

![PJ02-SS04](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/65614235-144d-45b4-989c-47d4f57d3283)

<strong>If you pasted all the generated JSON file into GitHub > Settings > Actions > Repo Secret. As done above, you shouldn't have any error after you click save on Deployment in your azure </strong> </br>

### Step 07: Resolve Build Errors(Frontend App)
<p>If you encounter build errors during frontend app deployment, make the necessary changes in the package.json file as shown below, then run </br>
<code>npm install</code>
  <code>npm audit fix --force</code> locally, and modify the package.json file as shown in the image below commit neccessary changes and push the changes to GitHub. This is fix the build error on the frontendapp and should automatically deploy to azure on every git push
</p>

  ![PJ02-SS05](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/20518d46-6e8a-4fd4-bbfb-a7f1571d2575)

<p>Here is a link to my Repos as issues already fixed here</p>
<ul>
  <li><a href="https://github.com/danieldgtal/nodejs-frontend">Frontend App(Repo)</a></li>
  <li><a href="https://github.com/danieldgtal/nodejs-backend">Backend App(Repo)</a></li>
</ul>

### Step 08: Validate connections and app access

![PJ02-SS06](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/74fdde1a-bfa7-4455-aa50-b466889ad80b)

![PJ02-SS07](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/21065323-0d69-4717-b5f1-ca066ff16bae)

![PJ02-SS08](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/7fc3bc68-cab9-4188-9641-d33e4951014c)

<p>The <code>nslookup</code> returns the public IP for the back-end web app. Since public access to the back-end web app is disabled, if you try to reach the public IP, you get an access denied error. This error means this site isn't accessible from the public internet, which is the intended behavior. The <code>nslookup</code> doesn’t resolve to the private IP because that can only be resolved from within the virtual network through the private DNS zone. Only the front-end web app is within the virtual network. If you try to run <code>curl</code> on the back-end web app from the external terminal, the HTML that is returned contains <code>Web App - Unavailable</code>. This error displays the HTML for the error page you saw earlier when you tried navigating to the back-end web app in your browser.</p>

### Step 09: Adding a cache layer 
