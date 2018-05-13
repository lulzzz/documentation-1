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

Create a file called 'identityservice.yaml' and add the following code. Replace the <<place holder text>> with the data you copied for the SQL Database and SendGrid Service earlier in this article. You will also need to include an Admin Email address.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: identitysvc
spec:
  template:
    metadata:
      labels:
        app: identitysvc
    spec:
      containers:
      - image: rcladmin/identitysvc3:125
        name: identitysvc
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: Production
        - name: DatabaseConnection
          value: Server=tcp:<<your-sql-server-name>>,1433;Initial Catalog=<<your-db-name>>;Persist Security Info=False;User ID=<<your-sql-server-username>>;Password=<<your-sql-server-password>>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;       
        - name: SendGridAPIKey
          value: <<your-sendgrid-api-key>>
        - name: AdminUserName
          value: <<your-admin-email>>
---
apiVersion: v1
kind: Service
metadata:
  name: identitysvc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: identitysvc
  type: ClusterIP        
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: identitysvc-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - <<your-dns-name>>
    secretName: acme-crt-secret
  rules:
  - host: <<your-dns-name>>
    http:
      paths:
      - path: /identitysvc
        backend:
          serviceName: identitysvc
          servicePort: 80
```

To install Identity Service, CD into the folder where you saved the file and run the following command

```
kubectl create -f identityservice.yaml
```

The ingress rules will allow the load balancer to route all requests with path /identitysvc to identity service. The site will also be run with HHTPS. 

Navigate to the site at https://<<your-dns-name>>/identitysvc

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/identity/identity-1.PNG "Image")



