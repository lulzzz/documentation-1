---
layout: category
title: 03 Install an Ingress Controller
---

An ingress controller is a piece of software that provides reverse proxy, configurable traffic routing. Kubernetes ingress resources are used to configure the ingress rules and routes for individual Kubernetes services. Using an ingress controller and ingress rules, a single external address can be used to route traffic to multiple services in a Kubernetes cluster.

## Install NGINX Ingress Controller with Helm

Update the Helm repository

```bash
helm repo update
```

Install the nginx ingress controller with RBAC disabled

```bash
helm install --name nginx-ing stable/nginx-ingress --set rbac.create=false --namespace kube-system
```

NGINX ingress controller should now be installed in your cluster. Azure AKS automatically adds a load balancer called Kubernetes and a public IP to the resource group.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/ingress/ingress-1.PNG "Image")

Run the following command to see the services running in the cluster in the kube-system namespace

```bash
kubectl get services --namespace kube-system
```

Output:

```bash
NAME                                      TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                      AGE
heapster                                  ClusterIP      10.0.195.50    <none>         80/TCP                       2h
kube-dns                                  ClusterIP      10.0.0.10      <none>         53/UDP,53/TCP                2h
kubernetes-dashboard                      ClusterIP      10.0.58.84     <none>         80/TCP                       2h
nginx-ing-nginx-ingress-controller        LoadBalancer   10.0.141.187   23.100.26.95   80:30459/TCP,443:31810/TCP   11m
nginx-ing-nginx-ingress-default-backend   ClusterIP      10.0.42.127    <none>         80/TCP                       11m
tiller-deploy                             ClusterIP      10.0.115.145   <none>         44134/TCP                    56m
```

You will see the nginx controller that can be accessed from the external internet (type: LoadBalancer) with an external IP assigned. You will also see the an nginx default backend that runs inside the cluster an cannot be accessed from the internet (type: ClusterIP). The Load Balancer will route traffick to the default backend when a frontend website is not present. If you navigate to the external IP address (in this instance, 23.100.26.95) you will see the nginx default backend in the browser.

![placeholder](https://raw.githubusercontent.com/rcl-microservices-aks/documentation/master/images/ingress/ingress-2.PNG "Image")

## Increasing the NGINX Proxy Buffer Size

The NGINX ingress controller Proxy Buffer Size is not large enough for the headers returned from some services. You will need to increase the buffer size. Create a ConfigMap file called 'nginx-configuration.yaml' and add this code. Replace the [placeholder] text with your nginx pod name.

```bash
kind: ConfigMap  
apiVersion: v1  
metadata:  
  name: [your-nginx-pod-name]
  namespace: kube-system
data:  
  proxy-buffer-size: "16k"
```

You can get the pod name for nginx with the following command

```bash
kubectl get pods --namespace kube-system
```

Output:
```bash
NAME                                                       READY     STATUS    RESTARTS   AGE
heapster-6599f48877-zj927                                  2/2       Running   0          2h
kube-dns-v20-7c556f89c5-5w5d2                              3/3       Running   0          2h
kube-dns-v20-7c556f89c5-fk8st                              3/3       Running   0          2h
kube-proxy-zksb6                                           1/1       Running   0          2h
kube-svc-redirect-4zr5z                                    1/1       Running   0          2h
kubernetes-dashboard-546f987686-v2g9v                      1/1       Running   0          2h
nginx-ing-nginx-ingress-controller-56b8ff549-6lshm         1/1       Running   0          27m
nginx-ing-nginx-ingress-default-backend-7f6dc6f7c4-dw5h4   1/1       Running   0          27m
omsagent-zt84m                                             1/1       Running   1          2h
tiller-deploy-5dd4f7b964-qz4rk                             1/1       Running   0          28m
tunnelfront-df6544c87-c6vv9                                1/1       Running   0          2h
```

The pod name in this instance is 

```bash
nginx-ing-nginx-ingress-controller-56b8ff549-6lshm
```

CD into the folder where you saved the ConfigMap and run the following command:

```bash
kubectl apply -f nginx-configuration.yaml 
```

The nginx ingress controller will be updated and restarted with the new configuration value for the proxy buffer size.

### Next Steps

The other article in this website will walk you through installing HTTPS certificates with cert-manager.

Next Article : [Install HTTPS Certificates](/category/04_install_cert_mgr) 



