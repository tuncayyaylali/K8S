# CloudNative-PG Setup and PostgreSQL Connection via DBeaver

This document walks you through the setup of CloudNative-PG on a Kubernetes cluster, exposing the PostgreSQL database externally, and connecting to it using DBeaver.

## Add the CloudNative-PG Helm Repository

First, add the official CloudNative-PG Helm chart repository to Helm:

```bash
# Add the CloudNative-PG Helm repository
helm repo add cloudnative-pg https://cloudnative-pg.io/charts/
```
```bash
# Update your local Helm repository
helm repo update
```
## Install CloudNative-PG using Helm
```bash
# Install CloudNative-PG using Helm
helm install cloudnative-pg cloudnative-pg/cloudnative-pg --version 0.22.0 -n cloudnative-pg --create-namespace --set service.type=LoadBalancer
```
```bash
# Apply the following YAML manifest to create a PostgreSQL cluster with 3 instances and 5Gi of storage for each.
cat <<EOF | kubectl apply -f -
# Example of a PostgreSQL cluster with 3 instances
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-example
spec:
  instances: 3
  storage:
    size: 5Gi
EOF
```
```bash
# Check the status of the PostgreSQL cluster
kubectl get cluster
```
```bash
# Expose the read-write service externally
kubectl expose svc cluster-example-rw --type=LoadBalancer --name=postgresql-loadbalancer --port=5432 --target-port=5432 -n default
```
```
NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)          AGE
cluster-example-r         ClusterIP      10.43.42.180    <none>           5432/TCP         37m
cluster-example-ro        ClusterIP      10.43.153.85    <none>           5432/TCP         37m
cluster-example-rw        ClusterIP      10.43.97.49     <none>           5432/TCP         37m
keycloak                  LoadBalancer   10.43.145.240   192.168.81.150   8080:32390/TCP   34h
kubernetes                ClusterIP      10.43.0.1       <none>           443/TCP          8d
postgresql-loadbalancer   LoadBalancer   10.43.36.143    192.168.81.152   5432:31147/TCP   6m41s
```
## Retrieve PostgreSQL Credentials
```bash
# Export the PostgreSQL password from the Kubernetes secret
export cloudnative_pg_password=$(kubectl get secret cluster-example-app -o jsonpath="{.data.password}" | base64 --decode)
```
```bash
# Export the PostgreSQL username from the Kubernetes secret
export cloudnative_pg_username=$(kubectl get secret cluster-example-app -o jsonpath="{.data.username}" | base64 --decode)
```
## Connect to PostgreSQL Cluster Using DBeaver
Step-by-Step Instructions for DBeaver:
- Open DBeaver and select New Database Connection.
- Choose PostgreSQL as the database type and click Next.
- Enter the following connection details:
    - Host: External IP of the postgresql-loadbalancer service (e.g., 192.168.81.152).
    - Port: 5432.
    - Database: postgres (or your default database name).
    - Username: The username retrieved from the secret ($cloudnative_pg_username).
    - Password: The password retrieved from the secret ($cloudnative_pg_password).
    - Check the box for Show all databases.
    - Test Connection to verify the connection to PostgreSQL.
- Once successful, click Finish to complete the connection.

Now you can manage your PostgreSQL cluster using DBeaver!