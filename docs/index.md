---
layout: index
title: Home
---

## Introduction 

Identity Service runs as a separate web application and provides Single-Sign-On (SSO) for ASP.NET Core applications using Open ID connect. It is hosted in a docker container in an Azure Container Services (AKS) Linux cluster.

It can be used as a gateway in a micro-services architecture to provide identity services and to secure web APIs. Identity Service uses the Client Credentials flow to protect ASP.NET Core Web APIs.

You can also use it to provide authorized access to pages in you web application such as "Admin" pages. It provides a simple mechanism to manage users, roles and claims for ASP.NET Core Identity.

All the tasks carried out in Identity Service are done through a simple and intuitive web UI dashboard. 

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/misc/identity-server.PNG "Identity Service")


