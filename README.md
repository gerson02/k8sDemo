# Kubernetes, also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications
## Source: https://kubernetes.io/

<img src="https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg" />

<img src="https://www.zdnet.com/a/img/resize/e6736e35f09bddeead7a9ad74233ce564ce59017/2017/05/08/af178c5a-64dd-4900-8447-3abd739757e3/docker-vm-container.png?auto=webp&width=1200" />

To do this you need to install the Azure CLI
You could also other ways like Azure PowerShell Scripts
or the Azure Portal

# Deploy the cluster

## 1. Create the resource group

<code> az group create --name k8sDemo --location eastus </code>

## 2. Create the cluster

<code> az aks create -g k8sDemo -n AKSDemoCluster --enable-managed-identity --node-count 1 --enable-addons monitoring --generate-ssh-keys </code>

## 3- To connect to the cluster, you need the kubectl client that you can install with this command:

<code> az aks install-cli </code>

## 4-Connect to the cluster

<code> az aks get-credentials --resource-group k8sDemo --name AKSDemoCluster </code>

If you prefer you can use a graphical UI to interact with the cluster
using a solution called "Lens"

# Deploy an app

## 1. Create an App

<code> dotnet new webapi --framework netcoreapp3.1 </code>

## 2. Run it and test it

<code> dotnet run </code>

## 3. Publish the app to generate the binaries

<code> dotnet publish --configuration Release </code>

## 4.Create the Dockerfile. This is the image specification

## 5. Create docker image

<code> docker build -t k8s-demo:v1 . </code>

## 6. List the image to confirm its creation

<code> docker images --filter=reference='k8\*' </code>

## 7. Run the image

<code> docker run --name k8s-demo --publish 8080:80 --detach k8s-demo:v1 </code>

This will enable API in EP in port 8080

## 8- Create an Azure Container Registry (ACR) in Azure

I used the Azure portal for this and name the ACR k8sdemoacrintelcom

## 9- Authenticate to ACR

<code> az acr login --name k8sdemoacrintelcom </code>

## 10- Push the image using ACR in this case using ACR Tasks. It's possible to use docker as well.

<code> az acr build --image "k8s-demo:v2" --registry k8sdemoacrintelcom . </code>

# Deploy to AKS

<p>
The following command will "attach" ACR and AKS together so the second call pull
images from the first. This could be also done at cluster creation time attaching
an AD service principal to the cluster and using a managed identity
</p>

<code> az aks update -n AKSDemoCluster -g k8sDemo --attach-acr k8sdemoacrintelcom </code>

## 1. Create an ingress controller

   <p>This controller will serve as entry point to the app
   and works in conjunction with the Ingress object to
   allow the api calls.
   </p>
   
<code> kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.1/deploy/static/provider/cloud/deploy.yaml </code>

#### Note: It's possible to use HELM for this as well

## 2. Execute sequencially each one of the files included in the folder "k8sdeploy" in the following order:
### a) Create Ingress
### b) Create service
### c) Create deployment

## 3. With the public address created by the Ingress Controller (Load Balancer), test the App.

# Delete Azure Resources

<p>To prevent unwanted charges, remove all created resources by removing the resource grouo</p>

<code> az group delete -g k8sDemo </code>

