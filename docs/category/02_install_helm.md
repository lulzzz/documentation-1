---
layout: category
title: 02 Install Helm on Windows
---

## Helm

Helm is an open-source packaging tool that helps you install and manage the lifecycle of Kubernetes applications

## Install Chocolatey

You can install Helm on Windows using Chocolatey. You can download and install Chocolatey from the following page:

[Install Chocately](https://chocolatey.org/install)

You can :
* Install with cmd.exe
* Install with PowerShell.exe

## Install Helm with Chocolatey

Run the following command to install Helm

```bash
choco install kubernetes-helm
```

## Install Tiller

Tiller, the server portion of Helm, typically runs inside of your Kubernetes cluster. But for development, it can also be run locally, and configured to talk to a remote Kubernetes cluster.

### In-Cluster Installation

Tiller requires security priveleges to run properly in an AKS cluster. You must create a 'cluster-admin' role to run Tiller under. Create a file called 'cluster-admin.yaml' and add the following code :

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: cluster-admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
```

CD into the folder where you created the file and run the following command to create the 'cluster' admin role.

```bash
kubectl create -f cluster-admin.yaml
```

Create a service account for tiller

```bash
kubectl create serviceaccount tiller --namespace kube-system
```

Create a file called 'rbac-config.yaml' and add the following code to configure Tiller

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

Bind tiller to the 'cluster-admin' role and service account by running the following command. You will need to CD into the folder that you saved the 'rbac-config.yaml' file to run this command.

```bash
kubectl create -f rbac-config.yaml
```

Install Tiller with the service account

```bash
helm init --service-account tiller
```

Tiller should now be installed in the cluster.


