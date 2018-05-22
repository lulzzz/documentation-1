---
layout: index
title: Home
---

## Introduction 

In this article we will walk through a demonstration application that is built with Microservices from the ground up. You can follow along by installing the various applications to understand how they work.

### Azure Kubernetes Services (AKS)

Microsoft Azure Kubernetes Services (AKS) is an orchestration service for the hosting of Microservices applications. The following in a desciption of the application hosted in AKS.

### Nodes

A node may be a VM on the cluster. Each node has the services necessary to run pods and is managed by the master components.

### Pods

A pod is a group of one or more docker containers , with shared storage/network, and a specification for how to run the containers. Pods are run in nodes.

### Cluster

The group of nodes , as a whole, is called a cluster. This is created in an AKS cluster.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/cluster.PNG "Image")


