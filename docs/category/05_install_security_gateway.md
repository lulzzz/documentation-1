---
layout: category
title: 05 Install Security Gateway
---

## Security Gateway With Identity Service

Identity Service runs as a separate web application and provides Single-Sign-On (SSO) for ASP.NET Core applications using Open ID connect. It is hosted in a docker container in an Azure Container Services (AKS) Linux cluster.

It can be used as a gateway in a micro-services architecture to provide identity services and to secure web APIs. Identity Service uses the Client Credentials flow to protect ASP.NET Core Web APIs.

You can also use it to provide authorized access to pages in you web application such as “Admin” pages. It provides a simple mechanism to manage users, roles and claims for ASP.NET Core Identity.

All the tasks carried out in Identity Service are done through a simple and intuitive web UI dashboard.

## Prerequisites

Identity Service requires the following:

* Azure SQL Database
* SendGrid Email service

### Azure SQL Database

Identity Service stores its data in a SQL Database. Install an Azure SQL Database in your azure account or use an existing database and copy the following for use later on : 

* SQL Server name
* SQL Server database name
* SQL Server login username
* SQL Server login password

For detailed instructions on creating an Azure SQL Database, refer to the  [Azure SQL Server Documentation](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-get-started-portal)

### SendGrid Email Service

Identity Service requires verification of emails when a user first registers. We use the SendGrid email service for this. Install SendGrid in your Azure account or use an existing installation and copy the **SendGrid API Key** for use later on. For detailed instructions, refer to the  [Azure Send Grid Documentation](https://docs.microsoft.com/en-us/azure/sendgrid-dotnet-how-to-send-email)

## Install Identity Service

Run the following command to install Identity Service. Replace the [placeholder text with your data]

```bash
helm install rcl-apps/identityservice --set dbServerName=[db-server-name] --set dbName=[dbname] --set dbUserName=[db-username] --set dbPassword=[db-password] --set sendgridKey=[sendgrid-api-key] --set adminEmail=[admin-email] --set host=[dns-name]
```

An example of the command with sample values is as follows :

```bash
helm install rcl-apps/identityservice --set dbServerName=contoso.database.windows.net --set dbName=contosodb --set dbUserName=contosoadmin --set dbPassword=dffrtwedsfd --set sendgridKey=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 --set adminEmail=contoso@mail.com --set host=contoso.eastus.cloudapp.azure.com
```

Navigate to the site at https://<<your-dns-name>>/identitysvc

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/identity/identity-1.PNG "Image")

### Next Steps

The other article in this website will walk you through installing Web API services in the AKS cluster.

Next Article : [Installing Web API Services](/category/06_api_services) 



