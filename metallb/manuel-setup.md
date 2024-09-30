# MetalLB Installation

This document walks through the installation of MetalLB, IP Address Pool and Layer 2 Advertisement configuration, and deploying an Nginx service using the LoadBalancer type on Kubernetes. 

## Installation of MetalLB

First, we install MetalLB using the official manifest provided on GitHub.

```bash
# Install MetalLB in your Kubernetes cluster
kubectl apply -f https://github.com/metallb/metallb/blob/v0.14/config/manifests/metallb-native.yaml
```
```bash
# Get all resources in the metallb-system namespace to verify MetalLB installation
kubectl get all -n metallb-system
```
```
NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-6dd967fdc7-gl74j   1/1     Running   0          12m
pod/speaker-4lfl5                 1/1     Running   0          12m
pod/speaker-c4vjz                 1/1     Running   0          12m
pod/speaker-nvl8r                 1/1     Running   0          12m

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/metallb-webhook-service   ClusterIP   10.43.162.107   <none>        443/TCP   12m

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   3         3         3       3            3           kubernetes.io/os=linux   12m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           12m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-6dd967fdc7   1         1         1       12m
```
## Creating IP Address Pool
```bash
# Create the IP address pool configuration file
sudo vi ipaddress-pool.yaml
```
```YAML
# Define the IP range that MetalLB can assign to services
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.81.150-192.168.81.199
```  
```bash
# Apply the IP address pool configuration
kubectl apply -f ipaddress-pool.yaml
```
```bash
# Verify that the IP address pool has been created
kubectl get ipaddresspools.metallb.io -n metallb-system
```
```
NAME         AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
first-pool   true          false             ["192.168.81.150-192.168.81.199"]
```
## Creating L2 Advertisement 
```bash
# Create the L2 advertisement configuration for the IP address pool
sudo vi l2advertisement-first-pool.yaml
```
```YAML
# Configure L2 advertisement for the IP address pool
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
  - first-pool
```
```bash
# Apply the L2 advertisement configuration for the IP address pool
kubectl apply -f l2advertisement-first-pool.yaml
```
```bash
# Verify that the L2 advertisement for the IP address pool is applied
kubectl get l2advertisements.metallb.io -n metallb-system
```
```
NAME      IPADDRESSPOOLS   IPADDRESSPOOL SELECTORS   INTERFACES
example   ["first-pool"]
```
## Nginx 
```bash
# Create a new namespace for Nginx
kubectl create ns nginx
```
```bash
# Deploy Nginx in the nginx namespace
kubectl create deploy nginx --image nginx -n nginx
```
```bash
# Expose the Nginx deployment as a LoadBalancer service
kubectl expose deploy nginx -n nginx --port 80 --type LoadBalancer
```
```bash
# Verify that the LoadBalancer service has been created
kubectl get all -n nginx 
```
```
NAME    TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
nginx   LoadBalancer   10.43.166.128   192.168.81.151   80:32731/TCP   10s
```
```bash
# Test the external IP of the Nginx service
curl 192.168.81.151
```
```HTML
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
This markdown file includes explanations and relevant code sections, categorized by type (bash for commands, yaml for configuration files), making it easier to follow the entire process step by step.
