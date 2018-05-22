---
layout: index
title: Home
---

## Introduction 

In this series of articles, we will walk through a demonstration application that is built with Microservices from the ground up. You can follow along by installing the various applications to understand how they work together.

### Azure Kubernetes Services (AKS)

Microsoft Azure Kubernetes Services (AKS) is an orchestration service for the hosting Microservices applications. The following is a description of the demonstration applications hosted in AKS.

### Nodes

A node is a Virtual Machine (VM) in the cluster. Each node has the services necessary to run pods and is managed by a master component.

### Pods

A pod is a group of one or more docker containers, with shared storage/network, and a specification for how to run the containers. Pods are run in nodes.

### Cluster

The group of nodes , as a whole, is called a cluster. The following diagram illustrates our Azure Kubernetes Service (AKS) cluster.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/cluster.PNG "Image")

### Services

The main microservices that make up the application are:

* Product Service - This is an ASP.NET Core Web API that manages product data.
* Order Service - This is an ASP.NET Core Web API that manages order data.
* Identity Service = This is an ASP.NET MVC application that provides single-sign-on for the frontend website and provides authorization services for the Web APIs.
* Store Front End = This is an ASP.NET Core MVC application that will serve as our store front end website.

### High Availability

Gor high availability the AKS cluster automatically assigns services to pods and nodes. Later on we can see how requests are routed to services within the pods to load balance the requests.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/cluster-2.PNG "Image")


