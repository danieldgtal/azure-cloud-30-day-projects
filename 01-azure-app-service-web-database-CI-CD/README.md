# 01 - Dive into a specific cloud service you're less familiar with and explore its features.
This guide is part of the CozyCloud Project challange, where i took ownership of implementing the project idea From Conception to
Completion. While the initial idea did not originate from me, I executed the ideas meticulously for project based purpose and this piece
is my process documentation. 
## Introduction
In This section, I delve into deploying an azure app service. Web + Database and configured a CI/CD pipeline using github action.
A simple laravel app was forked from a repo and pull to my local machine tested and works locally and deploy to azure.
## Prerequisite
. Microsoft Acount \
. Github account \
. VSCode 
## Azure services used
* Azure App Service
* Azure Service Principal
## Setup your environment
We’ll begin to set up our environment both for local development (Optional) and for Azure.

### 00 Forking the Laravel Application
Forking the laravel sample app repo clone https://github.com/Azure-Samples/laravel-tasks.git
![PJ01-SS01](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/88d243eb-9250-4d23-bc7a-1df13c65a853)
If you want to run the application locally, do the following:
<ul>
  <li>In .env, configure the database settings (like DB_DATABASE, DB_USERNAME, and DB_PASSWORD) using settings in your local Azure Database for MySQL flexible server database. You need a local Azure Database for MySQL flexible server instance to run this sample.</li>
   <li>From the root of the repository, start Laravel with the following commands:</li>
  <code>composer install
  php artisan migrate
  php artisan key:generate
  php artisan serve</code>
  
  ![PJ01-SS02](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/2a63e3f3-24f9-4edd-95c1-a38c0df97958)
 
</ul>

### 01 Create an Azure App Service + Database
In this step, you provision the resources needed to deploy our Laravel Application
<ul>
  <li>Login to the azure portal and search for and select App Services. Next select create then webapp + Database and fill out the neccessary forms. which are Resourece groups, region, name, runtime stack,etc. note the database name, username and password at the review + Create page and click on create.</li>
  <br>

  ![PJ01-SS03](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/865403e4-439f-4d5c-8523-ffc05d889385)

  ![PJ01-SS04](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/0a9bb809-c02e-4861-a348-8a5224ef7354)

  <strong>
    Refer to this microsoft documentation for more information on provisioning the resource.
    Note you have to come back and follow this guide to the end to be able to configure CI/CD which is where registering app service comes in else you'll be stuck with error following the outdated mslearn documentation.
    Follow through from <br>
      step 1- Create App Service  and Azure Database to <br>
      step 2- Set up database connectivity  <br>
      <link>https://learn.microsoft.com/en-us/azure/mysql/flexible-server/tutorial-php-database-app</link>
  </strong>  
  </br>
  <li>
    Select the settings blade and under the application settings tab, your configuration should now look like the image below
  </li>
  <br>
  
  ![PJ01-SS05](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/c6c2b3eb-d84b-4da5-b607-b6bf7fc995bd)
  
  <li>
    <tr>
      <td style="text-align: left;">Create a new <code>MYSQL_ATTR_SSL_CA</code> database setting:
        <ol>
          <li>
            <p>Click <strong>New application setting</strong>.</p>
          </li>
          <li>
            <p>In the <strong>Name</strong> field, enter <em>MYSQL_ATTR_SSL_CA</em>.</p>
          </li>
          <li>
            <p>In the <strong>Value</strong> field, enter <em>/home/site/wwwroot/ssl/DigiCertGlobalRootCA.crt.pem</em>.</p>
          </li>
          <li>
            <p>Click <strong>OK</strong>.</p>
          </li>
        </ol>
      </td>
    </tr>
  </li>
  <li>
   <td style="text-align: left;">Create the following extra app settings by following the same steps, then click on <strong>Save</strong>.
      <ul>
      <li>
        <p><em>APP_DEBUG</em>: Use <em>true</em> as the value. This is a <a href="https://laravel.com/docs/8.x/errors#configuration" data-linktype="external">Laravel debugging variable</a>.</p>
      </li>
      <li>
        <p><em>APP_KEY</em>: Use <em>base64:Dsz40HWwbCqnq0oxMsjq7fItmKIeBfCBGORfspaI1Kw=</em> as the value. This is a <a href="https://laravel.com/docs/8.x/encryption#configuration" data-linktype="external">Laravel encryption variable</a>.</p>
      <div class="alert is-primary">
      <p class="alert-title"><span class="docon docon-status-info-outline" aria-hidden="true"></span> Important</p>
      <p>This <code>APP_KEY</code> value is used here for convenience. For production scenarios, it should be generated specifically for your deployment using <code>php artisan key:generate --show</code> in the command line.</p>
      </div>
      </li>
      </ul>
    </td>
  </li>
  <li>
    <p id="database">In Visual Studio Code either on your local machine or github, open config/database.php. In the mysql connection driver, see that the app settings are same as configured on azure. Finally commit.Lastly push if you are on your local git.</p>
  </li>

  ![PJ01-SS06](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/0596867f-9dfe-49c5-8fe7-bef553fe4d4b)
</ul>

### 02 Create an Azure Service Principal
An Azure service principal is an identity created for use with applications, hosted services, and automated tools to access Azure resources. There are other ways to create an identity on azure to be able to authenticate and authorize github with azure but for this purpose we will use Azure Service Principal
<ol>
  <li>
    <p>Open Azure Cloud Shell in the Azure portal or Azure CLI locally.</p>
  </li>
  <li>
    <p>Create a new service principal in the Azure portal for your app. The service principal must be assigned with an appropriate role.</p>
    <code>
      az ad sp create-for-rbac --name "myApp" --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} --json-auth
    </code>
    <p>The parameter --json-auth outputs the result dictionary accepted by the login action, accessible in Azure CLI versions >= 2.51.0. Versions prior to this use --sdk-auth with a deprecation warning.</p>
  </li>
  <li>
    <p>Copy the JSON object for your service principal.</p>
    <code>
      {
      "clientId": "<GUID>",
      "clientSecret": "<GUID>",
      "subscriptionId": "<GUID>",
      "tenantId": "<GUID>",
      (...)
      }
    </code>
  </li>
</ol>
### 03 Add the Service Principal as a GitHub secret
<ol>
  <li><p>In GitHub, go to your repository.</p></li>
  <li><p>Go to Settings in the navigation menu.</p></li>
  <li><p>Select Security > Secrets and variables > Actions.</p></li>
  <li><p>Select New repository secret.</p></li>
  <li><p>Paste the entire JSON output from the Azure CLI command into the secret's value field. Give the secret the name AZURE_CREDENTIALS.</p></li>
   <li><p>Select add a secret</p></li>
</ol>

![PJ01-SS07](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/6cfdb9e1-517a-4e07-bdc0-943d9c706508)

<p>For more information on azure service principal visit <link>https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure?tabs=azure-portal%2Cwindows#use-the-azure-login-action-with-a-service-principal-secret</link></p>

### 04 Setup CI/CD Pipeline Using Github Action and Deploy Code
<p>In this step, you'll configure GitHub deployment using GitHub Actions. It's just one of the mant ways to deploy to App Service, but a great way to have continous integration in your deployment process. By Default every push to GitHub repo will kick off build and deploy action</p>
<p>In Visual Studo Code, open config/database.php and ensure the database driver settings is properly setup. Refer to the database connection section <a href="#database">setup database</a> </p>
<br>
<p>Back in the App Service page, in the left menu, select Deployment Center.</p>
<ol>
  <p><strong>In the Deployment Center page</strong></p>
  <li>In Source, select GitHub. By default, GitHub Actions is selected as the build provider.</li>
  <li>Sign in to your Github account and follow the prompt to authorize Azure</li>
  <li>In Organization, select your account</li>
  <li>In Repository, select the repository name</li>
  <li>In Branch, select main</li>
  <li>In the top menu, click Save.</li>
</ol>
<p>App Service commits a workflow file into the selected GitHub repository, in the .github/workflows directory.</p>

![PJ01-SS08](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/1bde5fd2-bb37-4b63-8c3a-f9fd201d40a3)

<p>Refer to section 3 of this MSlearn docs for more <a href="https://learn.microsoft.com/en-us/azure/mysql/flexible-server/tutorial-php-database-app">info </a></p>

In the Deployment Center page:
<ul>
  <li>Select Logs. A deployment is already started.</li>
  <li>In the log item for the deployment run, select <strong>Build/Deploy Logs.</strong></li>

  <p>You're taken to your GitHub repository and see that the GitHub action is running. The workflow file defines two separate stages, build and deploy.</p>
</ul>

![PJ01-SS09](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/89ce7978-a945-437a-907e-72b3b88c1a30)

### 05 Generate database schema 
<p>The creation wizard puts the Azure Database for MySQL flexible server instance behind a private endpoint, so it's accessible only from the virtual network. Because the App Service app is already integrated with the virtual network, the easiest way to run database migrations with your database is directly from within the App Service container.</p>

![PJ01-SS10](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/97ab893b-6f60-4b5e-831f-f8a0043d5b3b)

![PJ01-SS11](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/1a3b6320-5102-48f0-809f-3f0fe7fd366f)

### 06 Change site root

![PJ01-SS12](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/09a04f0f-3e54-4d10-87e6-80fbfeeb2116)

### 07 Browse to the app 

![PJ01-SS13](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/7219419a-be13-4cfe-900b-942a3d074d3e)

### 08 Stream diagnostic logs

![PJ01-SS14](https://github.com/danieldgtal/azure-cloud-30-day-projects/assets/51647363/e8961e78-ee94-4615-8fac-d78a69b8dfbb)

### 09 Clean up resources
<p>When you're finished, you can delete all of the resources from your Azure subscription by deleting the resource group</p>
