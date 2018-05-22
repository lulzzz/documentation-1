---
layout: index
title: Home
---

## Introduction 

In this series of articles, we will walk through a demonstration application that is built with Microservices from the ground up. This section will provide an overview of designing a Microservices application by describing how we created the demo application. Later on, you can try installing the various applications and exploring their features to understand how they work together.

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

### Load Balancer and Ingress Controller

Requests to the demo application is routed through a Load Balancer. An ingress controller is responsible for routing requests to the services. There is one external public IP address and DNS name for the application. For instance, the public IP for the cluster may be : '23.100.26.95' and the DNS may be : 'rclappdev.eastus.cloudapp.azure.com'. In the demo, requests are routed based on paths. As an example, a request with a path '/store' (https://rclappdev.eastus.cloudapp.azure.com/store) will be routed to the store frontend website. 

The communication between the web frontend and the APIs is also done through the ingress controller. For instance, a request from the fronted website to the Product Service using the path '/productsvc' (https://rclappdev.eastus.cloudapp.azure.com/productsvc) will be routed to the Product Service Web API.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/cluster-4.PNG "Image")

### HTTPS Termination

The Ingress Controller is also responsible for the TLS termination of all requests. HTTPS certificates are provided for the frontend website and each service by the cert-manager service that generates HTTPS certificates via the 'Let's Encrypt' project.

### Authentication and Authorization Services

#### Single Sign On

Identity Service provides Single-Sign-On (SSO) for the frontend website. When the user clicks the 'Login; link in the website, they are signed in using a login web page provided by Identity Service. Apart from the user authentication, Identity Service will provide the frontend website with a set of authentication claims that are assigned to the user. These authentication claims can be used to gain access to various operations on the website, for example, access to a membership or admin page.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/sso.PNG "Image")

#### Authorized API Access

Identity Service also provides security access to the Web APIs. The steps for securing an API are as follows:

* Request a Token - The frontend website requests a JWT access token from Identity Service to make a secured request to an API
* Provide a Token - Identity service will send the access token to the frontend website once it provides the right client id and secret to access the token
* Make and authenticated request - The frontend website will place the bearer token in the header and make a secured request to the API
* Return JSON Payload - Once the request is authorized by the API, the API will return the JSON payload to the frontend website


![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/api-auth.PNG "Image")

### Communication Between API Services

APIs can communicate with each other using REST or asynchronous messaging. The demo application uses Azure Service Bus messaging services for communication. The following example will illustrate the concept of messaging. 

The Product Service tracks the inventory of items with an 'inStock' field in the database. Each time an order is created by the Order Service, the Product Service will reduce the 'inStock' value of the product by one. So if there are 1000 pencils in stock, and an order is created for a pencil, the 'inStock' value for pencils is reduced to 999.

The Order Service can send a request directly to the Product Service via REST to update the 'inStock' value of the product in the database. However, this will create a dependency between the Order Service and the Product Service. A better approach would be to use a broker messaging service to accomplish this communication.

We can use 'Topics' in Azure Service Bus to create a Publish/Subscribe messaging service. The Order Service will publish a message to a Topic called 'Created-Order' each time an order is created. The message will be a simple JSON string containing the product id and other fields related to the order. Now, any service can subscribe to this Topic. The Product Service will subscribe to the Topic, and each time an order is created, the JSON string message will be transmitted to the Product Service asynchronously. The Product Service will then use the product id field in the JSON to find the product that was ordered and then reduce the 'inStock' value of the product by one.

You will observe that the Order Service does not know anything about the Product Service as it is only aware of Service Bus. Similarly, the Product Service is only aware of the Service Bus and not the Order Service. In this way, both services are completely de-coupled.

This approach also makes our application extensible. Say we now have a Shipping Service. The shipping service can subscribe to the 'Created-Order' Topic, and each time a new order is created it can send out an instruction to have the order shipped to a customer address.

Messaging also adds reliability to the application. For instance, if the Product Service is down, the message will remain in the Topic queue until the Product Service is back up again to consume it.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/intro/messaging.PNG "Image")

### Next Steps

The other articles in this website will walk you through installing the demo application and exploring the concepts we introduced above in greater detail.

