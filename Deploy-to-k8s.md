In this instruction we will learn how to deploy a spring boot backend to kubenertes. There are 4 important parts you need to follow.
1. Create resource group( this resource like a project that will content other thing like k8s, database, container registry)
2. Create azure [postgres database](https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-server-database-azure-cli)
3. Create [container registry](https://docs.microsoft.com/bs-cyrl-ba/azure/container-registry/container-registry-quickstart-task-cli) (where we store our images for deployment to k8s)
4. Create [k8s cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough) (where we can run and expose our application to the internet)
5. Create [public static ip](https://docs.microsoft.com/en-us/azure/aks/static-ip) for k8s(we will have a fixed ip for get things easier)

We keep them simplier in the following instruction. Now let's get started


# CREATE RESOURCE

## Create resource
`az group create --name myDevResource --location southeastasia`
This command will create a new resource group in south east asia.
> For other location code run this command below
>
> `az account list-locations`
>
> If you want do delete resource group use this command
>
> `az group delete --name myDevResource`
>
> With this command everything under this resource will be deleted too such as k8s, database, container register...


# CREATE AZURE POSTGRESQL

We will create a azure postgres database [Reference link](https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-server-database-azure-cli)


## 1. Create server
We will create a new postgres server with this command. 

`az postgres server create --resource-group myDevResource --name mydevdaydb  --location southeastasia --admin-user devdayadmin --admin-password Devd@y-2@19 --sku-name B_Gen5_1 --version 9.6`
> See this link for more information about [Pricing tiers](https://docs.microsoft.com/en-us/azure/postgresql/concepts-pricing-tiers) 

## 2. Config file rule allow IP connect
In order to allow remote connect to this database server we will set ip range for it with this command below

`az postgres server firewall-rule create --resource-group myDevResource --server mydevdaydb --name AllowMyIP --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255`

> To get detail information of this database try this comamnd
> 
> `az postgres server show --resource-group myDevResource --name mydevdaydb`

## 3. Connect to server via psql and create first database

`psql -U devdayadmin@mydevdaydb -d postgres -h mydevdaydb.postgres.database.azure.com`

> There are some command you can try with psql or [cheat sheet](http://www.postgresqltutorial.com/postgresql-cheat-sheet/)
>
> \l list all database
>
> \c chose database

### 3.1 Create db
`CREATE DATABASE devday;`

### 3.2 Install pgcryto extension
To use **digest function in sql** we have to install the following extension. We have to **select devday database** that you want to install this extension.
> 
`create extension pgcrypto;`
> 
> List all extension from posgres
>
> `SELECT * FROM pg_available_extensions;`

# CREATE CONTAINER REGISTER

## 1. Create container registry
We will create a container registry name myDevCR with this command

`az acr create --resource-group myDevResource --name myDevCR --sku Basic`

## 2. We clone this backend code

`git clone https://github.com/DevDay2019-AxonActive/DevDay2019-Performance-Tuning`

Then we will use this docker file [docker file](https://raw.githubusercontent.com/DevDay2019-AxonActive/DevDay2019-DevOps/Prod/backend/Dockerfile) for building and pushing image to container

## 3. Build And Push Image
`az acr build -t devday-backend:latest --registry myDevCR --file Dockerfile .`

> One single command for getting, build and push iimage
>
> `wget -O Dockerfile https://raw.githubusercontent.com/DevDay2019-AxonActive/DevDay2019-DevOps/Prod/backend/Dockerfile | az acr build -t devday-backend:latest --registry myDevCR --file Dockerfile .`
>
> Some helpful commands
>
> `az acr list` #List all container registry 
>
> `az acr repository list --name myDevCR` #List all images/repository
>
> `az acr repository show-tags --repository devday-backend --name myDevCR` #List all version of image/repository


# CREATE K8S
## 1. register provider
`az provider register --namespace Microsoft.Network`

## 2. create k8s
`az aks create --resource-group myDevResource --name myK8s --node-count 1 --enable-addons monitoring --generate-ssh-keys --location southeastasia`

## 3. Connect k8s to container register for pulling image and deploy it
`az aks update -n myK8s -g myDevResource --attach-acr myDevCR`

# DEPLOY TO K8S

After creating cluster we can create a yaml file for deployment

## 1. deploy with deployment
`kubectl apply -f https://raw.githubusercontent.com/DevDay2019-AxonActive/DevDay2019-DevOps/master/backend/Deployment.yaml`


> Some command for troubleshooting
> `kebectl get pods` #Get all pod that running  
> `kubectl describe pod [podname]` #Check log of pod
> `kubectl logs pod [podname]` #Check log of application(container)

# CREATE STATIC IP

Everytime we expose container to the internet k8s will reserve a different ip for you. So that in case we want to have a static ip for every time deploy we can this these steps below

## 1. Get NodeResourceGroup
This static ip must be in the same group of node so that we have to get node resource group first.

`az aks show --resource-group myDevResource --name myK8s --query nodeResourceGroup -o tsv`

## 2. Create static public ip
> note: public ip must be in standard sku

`az network public-ip create --resource-group MC_myDevResource_myK8s_southeastasia --name myAKSPublicIP --allocation-method static --sku Standard`
 
> `az network public-ip list` #Show all static public ip
