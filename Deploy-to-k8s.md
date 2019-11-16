# CREATE RESOURCE

## List all location for create resource
`az account list-locations`

## Create resource
`az group create --name myDevResource --location southeastasia`

## Clean up resource
This command line will clean up everything under this resource like k8s, database, container register...
`az group delete --name myDevResource`


# CREATE AZURE POSTGRESQL
https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-server-database-azure-cli
https://docs.microsoft.com/en-us/azure/postgresql/concepts-pricing-tiers

## Create server
`az postgres server create --resource-group myDevResource --name mydevdaydb  --location southeastasia --admin-user devdayadmin --admin-password Devd@y-2@19 --sku-name B_Gen5_1 --version 9.6`

## Config file rule allow IP connect
`az postgres server firewall-rule create --resource-group myDevResource --server mydevdaydb --name AllowMyIP --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255`

## Get connection
az postgres server show --resource-group myDevResource --name mydevdaydb

## psql connect for testing
`psql -U devdayadmin@mydevdaydb -d postgres -h mydevdaydb.postgres.database.azure.com`

## Create db
`CREATE DATABASE devday;`

## Clean up azure postgres
`az postgres server delete --resource-group myDevResource --name mydevdaydb`

## List all extension from posgres
`SELECT * FROM pg_available_extensions;`

## Install pgcryto extension
`create extension pgcrypto;`

# CREATE CONTAINER REGISTER

## Create container registery
`az acr create --resource-group myDevResource --name myDevCR --sku Basic`

## Build And Push Image
`az acr build -t devday-backend:v1 --registry myDevCR --file Dockerfile .`

## Build image from docker file in githup
`wget -O Dockerfile https://raw.githubusercontent.com/DevDay2019-AxonActive/DevDay2019-DevOps/Prod/backend/Dockerfile | az acr build -t devday-backend:v1 --registry myDevCR --file Dockerfile .`

## List all container registry
`az acr list`

## List all images/repository
`az acr repository list --name myDevCR`

## List all version of image/repository
`az acr repository show-tags --repository devday-backend --name myDevCR`

# CREATE K8S
## register provider
`az provider register --namespace Microsoft.Network`

## create k8s
`az aks create --resource-group myDevResource --name myK8s --node-count 1 --location southeastasia`

## Get credential
`az aks get-credentials --resource-group myDevResource --name myK8s`


# Create Secret key for pull image in deployment
## Enable admin first
`az acr update -n myDevCR --admin-enabled true`

## Show credential
`az acr credential show -n myDevCR`

## Create a Secret to hold the registry credentials.
`kubectl create secret docker-registry devday-secret-docker-register --docker-server=mydevcr.azurecr.io --docker-username=myDevCR --docker-password=NgfU4sw1lYHKKxTjRgO+oUuqhcQApPtO`

- username and password from ## Show credential

```
  imagePullSecrets:

- name: devday-secret-docker-register
```

> Note: there is a alternative way to pull image from k8s just run the following command so that we dont need.

`az aks update -n myK8s -g myDevResource --attach-acr myDevCR`

# DEPLOY TO K8S

## deploy with deployment
`kubectl apply -f Deployment.yaml`
`kubectl apply -f https://raw.githubusercontent.com/DevDay2019-AxonActive/DevDay2019-DevOps/master/backend/Deployment.yaml`

## check log pod
`kebectl get pods`
`kubectl describe pod backend-deployment-5b66cdd474-6vnrt`

## check log container
`kubectl logs pod backend-deployment-5b66cdd474-6vnrt`

# CREATE STATIC IP
## Get NodeResourceGroup
`az aks show --resource-group myDevResource --name myK8s --query nodeResourceGroup -o tsv`

## Create static public ip
`az network public-ip create --resource-group MC_myDevResource_myK8s_southeastasia --name myAKSPublicIP --allocation-method static --sku Standard`

> note: public ip must be in standard sku

## Show static public ip
`az network public-ip list`
