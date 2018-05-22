---
layout: category
title: 04 Install HTTPS Certificates
---

We will use the 'cert-manager' application to provide HHTPS certificates to our web facing services. cert-manager is used to automatically generate and configure 'Let's Encrypt' TLS certificates.

## Configure DNS name

Because HTTPS certificates are used, you need to configure an FQDN name for the ingress controllers IP address. In the AKS resource group open the Public IP address.


![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/ingress/ingress-1.PNG "Image")

Add a DNS name, in this instance we used 'rclappdev', but you must use your own unique name.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/cert/cert-1.PNG "Image")

## Install cert-manager

Use the following command to install cert-manager with RBAC disabled

```bash
helm install --name cert-mgr stable/cert-manager --set rbac.create=false
```

### Set a ClusterIssuer

ClusterIssuers represent a certificate authority from which signed x509 certificates can be obtained. You will need at least one ClusterIssuer in order to begin issuing certificates within your cluster. We will use 'Let's Encrypt' as our Cluster Issuer. Cluster Issuers do not belong to a single namespace and can be referenced by Certificate resources from multiple different namespaces.

Create a file called 'cluster-issuer.yaml' and add the following code. Replace the [placeholder] text with your email address.

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v01.api.letsencrypt.org/directory
    email: [your-email-address]
    privateKeySecretRef:
      name: letsencrypt-private-key
    http01: {}
``` 

To add the ClusterIssuer to the cluster, CD into the folder that you saved the file and run the following command

```bash
kubectl create -f cluster-issuer.yaml
```

## Create Certificates

cert-manager has the concept of ‘Certificates’ that define a desired X.509 certificate. A Certificate is a resource that references a ClusterIssuer for information on how to obtain the certificate.

Create a file called 'certificate.yaml' and add the following code. Replace the [placeholder] text with your DNS name that we set earlier in this article.

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: acme-crt
spec:
  secretName: acme-crt-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - [your-dns-name]
  acme:
   config:
    - http01:
        ingressClass: nginx
      domains:
      - [your-dns-name]
```

To create the certificate, CD into the folder where you saved the file and run the following command

```bash
kubectl create -f certificate.yaml
```

To get the details of the certificate, run the following command

```bash
kubectl describe certificate acme-crt
```

In the 'Events' section you should see

```bash
Events:
  Type     Reason                 Age   From                     Message
  ----     ------                 ----  ----                     -------
  Warning  ErrorCheckCertificate  1m    cert-manager-controller  Error checking existing TLS certificate: secret "acme-crt-secret" not found
  Normal   PrepareCertificate     1m    cert-manager-controller  Preparing certificate with issuer
  Normal   PresentChallenge       1m    cert-manager-controller  Presenting http-01 challenge for domain rclappdev.eastus.cloudapp.azure.com
  Normal   SelfCheck              1m    cert-manager-controller  Performing self-check for domain rclappdev.eastus.cloudapp.azure.com
  Normal   ObtainAuthorization    1s    cert-manager-controller  Obtained authorization for domain rclappdev.eastus.cloudapp.azure.com
  Normal   IssueCertificate       1s    cert-manager-controller  Issuing certificate...
  Normal   CeritifcateIssued      0s    cert-manager-controller  Certificated issued successfully
  Normal   RenewalScheduled       0s    cert-manager-controller  Certificate scheduled for renewal in 1438 hours
```

Ensure the certificate was issued successfully and the certificate is scheduled for automatic renewal within a 3 month period.

Navigate to the DNS name using https, you should now see a secured HTTPS connection

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/cert/cert-2.PNG "Image")


### Next Steps

The other article in this website will walk you through installing Identity Service as a Security Gateway.

Next Article : [Install Security Gateway](/category/05_install_security_gateway) 

