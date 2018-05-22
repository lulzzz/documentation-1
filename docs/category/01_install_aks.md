---
layout: category
title: 01 Create Azure Kubernetes Service
---

## Create AKS cluster

In this section we will create a simple one node AKS cluster using a B2S size VM for our node.

Choose Create a resource > search for Kubernetes > select Kubernetes Service  > Create.

The following images shows a sample configuration for the installation that will be used for this article.

### Basic

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/aks/install-1.PNG "Image")
![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/aks/install-2.PNG "Image")

### Networking
![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/aks/install-3.PNG "Image")

### Monitoring

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/aks/install-4.PNG "Image")

### Tags

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/aks/install-5.PNG "Image")

### Review and Create

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/aks/install-6.PNG "Image")

### Install Azure CLI on Windows

We will use the Azure CLI to manage our AKS Cluster. On Windows, the Azure CLI binary is installed via a MSI, which gives you access to the CLI through the Windows Command Prompt (CMD) or PowerShell.

You can download and install the CLI on the following page:

[MSI Installer](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)

You can now run the Azure CLI with the az command from either Windows Command Prompt or PowerShell.

```bash
az login
```

### Install the kubectl CLI

Kubectl is used to manage kubernetes. To install it locally, run the following command:

```bash
az aks install-cli
```

### Connect with kubectl

To configure kubectl to connect to your Kubernetes cluster, run the following command (replace the [placeholder] text with your data):

```bash
az aks get-credentials --resource-group [myResourceGroup] --name [myAKSCluster]
```

To verify the connection to your cluster, run the kubectl get nodes command.

```bash
kubectl get nodes
```

Output:

```bash
NAME                       STATUS    ROLES     AGE       VERSION
aks-agentpool-35064155-0   Ready     agent     17m       v1.9.6
```

### Next Steps

The other article in this website will walk you through installing the Helm package manager in our AKS cluster.

Next Article : [Installing Helm On Windows](/category/02_install_helm) 
















