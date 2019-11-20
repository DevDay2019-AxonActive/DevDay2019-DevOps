# download cli
https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest

# login
After installation azure cli client we have to login with the following command.

`az login`

This example will follow [this documentation](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-azure-cli)

# Create a container registry to store our images
This command line require you already had resource group if you dont have that resource group you can create it by this command below before creating container registry

`az group create --name [myDevResource] --location southeastasia`

Then you can create your container registry

`az acr create --resource-group [myDevResource] --name [myContainerRegistry] --sku Basic`

Then you have to login to container registry with command below

`az acr login --name [myContainerRegistry]`

Let's create a simple hello-world image or you can pull it public repository

`docker pull hello-world`

Then you have to tag this image with your azure container registry like

`docker tag hello-world <acrLoginServer>/hello-world:v1`
> <acrLoginServer> you can get this information get `az acr list`

Then you can push this image to registry by command docker push

`docker push <acrLoginServer>/hello-world:v1`

# Create k8s

`az aks create --resource-group [myDevResource] --name [myAKSCluster] --node-count 1 --enable-addons monitoring --generate-ssh-keys`

Connect k8s to container registry for access image from container registry

`az aks update -n [myAKSCluster] -g [myDevResource] --attach-acr [myContainerRegistry]`

Insctall kubectl 
`az aks install-cli`

Get credential for using kubectl cli

`az aks get-credentials --resource-group [myDevResource] --name [myAKSCluster]`

