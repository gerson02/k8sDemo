To do this you need to install the Azure CLI
You could also other ways like Azure PowerShell Scripts
or the Azure Portal

# Deploy the cluster

## 1. Create the resource group

az group create --name k8sDemo --location eastus

## 2. Create the cluster

az aks create -g k8sDemo -n AKSDemoCluster --enable-managed-identity --node-count 1 --enable-addons monitoring --generate-ssh-keys

## 3- To connect to the cluster, you need the kubectl client

that you can install with this command:
az aks install-cli

## 4-Connect to the cluster

az aks get-credentials --resource-group k8sDemo --name AKSDemoCluster
If you prefer you can use a graphical UI to interact with the cluster
using a solution called "Lens"

# Deploy an app

## 1. Create an App

dotnet new webapi --framework netcoreapp3.1

## 2. Run it and test it

dotnet run

## 3. Publish the app to generate the binaries

dotnet publish --configuration Release

## 4.Create the Dockerfile. This is the image specification

## 5. Create docker image

docker build -t k8s-demo:v1 .

## 6. List the image to confirm its creation

docker images --filter=reference='k8\*'

## 7. Run the image

docker run --name k8s-demo --publish 8080:80 --detach k8s-demo:v1
This will enable API in EP in port 8080

## 8- Create an Azure Container Registry (ACR) in Azure

I used the Azure portal for this and name the ACR k8sdemoacrintelcom

## 9- Authenticate to ACR

az acr login --name k8sdemoacrintelcom

## 10- Push the image using ACR (in this case using ACR Tasks. It's possible to use docker as well.

az acr build --image "k8s-demo:v2" --registry k8sdemoacrintelcom .

# Deploy to AKS

<p>
The follwing command will "attach" ACR and AKS together so the second call pull
images from the first. This could be also done at cluster creation time attaching
a AD service principal to the cluster and using a managed identity
</p>
az aks update -n AKSDemoCluster -g k8sDemo --attach-acr k8sdemoacrintelcom

## 1. Create an ingress controller
   This controler will serve as entry point to the app
   and works in conjunction with the Ingress object to
   allow the api calls.

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.1/deploy/static/provider/cloud/deploy.yaml

It's possible to use HELM for this as well

## 2. Execute sequencially each one of the files included in the folder k8s deploy in the following order:
### a) Create Ingress
### b) Create service
### c) Create deployment

