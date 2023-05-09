# Openshift Deployment

   We have to deploy 3 components for deployment of Eshop on web in Openshift.

      a) SQL server
      b) Public API
      c) Web App


   1) Clone this repo and change directory to repo cloned directory.

        ``` 
            git clone  https://github.com/arunhari82/dotnet-eShopOnWeb.git
            cd <<Repo directory Name>>  
        ```    

   2) Create a namespace for deploying the eshop application

            oc new-project <<namespace>>

   3) set the namespace as env variable

            export namespace=<<namespace>>        

## Prerequisite

## 1) SQL Server
   
   ### Deploying the sql server

   ### Execute following commands for deploying SQL server. 
   
   All these commands are available under `openshift/commands.txt` file

        oc new-project dotnet

        oc create secret generic mssql --from-literal=SA_PASSWORD="@someThingComplicated1234" -n $namespace
        oc create serviceaccount sqlserver-sa -n $namespace
        oc adm policy add-scc-to-user anyuid -z sqlserver-sa -n $namespace
        oc create -f openshift/sql-server/pvc.yaml -n $namespace
        oc create -f openshift/sql-server/Deployment.yaml -n $namespace
        oc expose deployment/sqlserver -n $namespace


 ## S2i Demo

 ### Public API.

        ## Create Configmap

        oc create -f openshift/publicApi/configmap.yaml

        ## S2i

        oc new-app dotnet:7.0-ubi8~https://github.com/arunhari82/dotnet-eShopOnWeb.git --name public-api --build-env DOTNET_STARTUP_PROJECT=src/PublicApi/PublicApi.csproj -e ASPNETCORE_URLS='http://+:5200' --strategy=source

        ### Wait for the build to complete before mounting configmap on deploymnet config

        oc set volume dc/public-api --add --name appsettings-vol --mount-path /opt/app-root/app/appsettings.json --configmap-name=appsettings-cm --sub-path=appsettings.json

  ### Web App. 

      oc new-app dotnet:7.0-ubi8~https://github.com/arunhari82/dotnet-eShopOnWeb.git --name web-app --build-env DOTNET_STARTUP_PROJECT=src/Web/Web.csproj -e ASPNETCORE_URLS='http://+:8080' --strategy=source
      
      oc set volume dc/web-app --add --name appsettings-vol --mount-path /opt/app-root/app/appsettings.json --configmap-name=appsettings-cm --sub-path=appsettings.json
      
      oc create route edge --service=web-app
          
 ## Docker Demo

 Install SQL Server which is a Prerequisite

 ### Public API :

 Use Import from Git from Console. Select the docker file as `src/PublicApi/Dockerfile`. Refer Screenshot below

 ![](/openshift/publicApi/assets/public-api-docker-statergy.png) 



        
    