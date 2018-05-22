---
layout: index
title: Home
---

## Introduction 

In this series of articles, we will walk through a demonstration application that is built with Microservices from the ground up. You can follow along by installing the various applications to understand how they work together.

### Azure Kubernetes Services (AKS)

Microsoft Azure Kubernetes Services (AKS) is an orchestration service for hosting Microservices applications. The following is a description of the demo applications hosted in AKS.

### Nodes

A node is a Virtual Machine (VM) in the cluster. Each node has the services necessary to run pods and is managed by a master component.

### Pods

A pod is a docker container, with shared storage/network, and a specification for how to run the containers. Pods are run in nodes.

### Cluster

The group of nodes , as a whole, is called a cluster. The following diagram illustrates our Azure Kubernetes Service (AKS) cluster.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/cluster.PNG "Image")

### Services

The main microservices that make up the demo application are:

* Product Service - This is an ASP.NET Core Web API that manages product data.
* Order Service - This is an ASP.NET Core Web API that manages order data.
* Identity Service - This is an ASP.NET MVC application that provides single-sign-on for the frontend website and provides authorization services for the Web APIs.
* Store Front End - This is an ASP.NET Core MVC application that will serve as our store front end website.

### High Availability

For high availability, the AKS cluster automatically assigns different instances of services to pods and nodes. Later on, we will see how requests are routed to services within the pods to load balance the requests.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/cluster-2.PNG "Image")

### Data 

The central theme in Microservices is the decoupling of each service. Services should handle a single business activity, and are usually small applications that can be developed by a few persons. Services should not have any strongly coupled dependency on the other services. Each service should be within a bounded domain as per the Domain-Driven-Design Model (DDD).

Each domain bounded services may have its own database. However, the demo application shares a common database for economic reasons. The tables for each service is independent of the tables for the other services. 

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/cluster-3.PNG "Image")

### Load Balancer

Requests to the demo application is routed through a Load Balancer. An ingress controller is responsible for roting requests to the services. There is one external IP address and DNS name. For instance the public IP for the demo cluster may be : 23.100.26.95 and the DNS may be : rclappdev.eastus.cloudapp.azure.com. In the demo, requests are routed based on paths. For instance a request with a path '/store' (https://rclappdev.eastus.cloudapp.azure.com/store) will be routed to the store web front end. 

The communication between the web frontend and the APIs is also done through the ingress controller. For instance a request from the fronted to the product service using the path '/productsvc' (https://rclappdev.eastus.cloudapp.azure.com/productsvc) will be routed to the Product Service Web API.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/cluster-4.PNG "Image")