# ReplicaSet

### Background

* selector.matchLabels needs to match pod labels
* apps/v1 for version
* content in spec.template is the same as as pod definition but without the **Kind** and **apiVersion** 

### Commands

Create replicaset

`kubectl apply -f replicaset.yml`

Get existing replicaset

`kubectl get replicaset`

`kubectl get rs`

Delete replicaset

`kubectl delete replicaset existing-replicaset`

Replace replicaset

`kubectl apply -f replicaset.yml`

Scale replicas to 3

`kubectl scale --replicas=3 -f replicaset.yml`

Scale existing replicas to 3

`kubectl scale --replicas=3 existing-replicaset`


# Deployment

Create deployment

`kubectl create deployment nginx --image=nginx`

Scale deployment to 3

`kubectl scale deployment nginx --replicas=3`


# Namespace

URL to another namespace `dev`, service `db-service`

`db-service.dev.svc.cluster.local`

* db-service is the name of service
* dev is the namespace
* svc is the k8 resource type - service


Create namespace dev

`kubectl create namespace dev`

View pods in dev namespace

`kubectl get pods --namespace=dev`

Stay in dev namespace

`kubectl config set-context $(kubectl config current-context) --namespace=dev`

View pods in all namespaces

`kubectl get pods --all-namespaces`

Specify ResourceQuota for your namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    # Maximum of 10 pods in the namespace
    pods: "10"
    # Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value.
    requests.cpu: "4"
    # Across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value.
    limits.cpu: "10"
    # Across all pods in a non-terminal state, the sum of memory limits cannot exceed this value.
    limits.memory: 10Gi
```

# Service

### Notes

* service use selector.label to determine its destination pod
* service has built-in loadbalancer to distribute loads across pods

### Nodeport

Service makes a port on the node accessible 

Port range: 30000 ~ 32767

### ClusterIP

In this case the service creates a virtual IP inside the cluster to enable communication between

different services such as a set of front end servers to a set of back end servers.


### Loadbalancer

Creates a LB in your cloud provider

# Imperative vs Declarative

### Imperative: A set of instructions written step by step

E.g:

1. Provision a VM
2. Install NGINX
3. Edit configuration file to use 8080
4. Edit conf file to use web path /var/www/nginx
5. Load web page to /var/www/nginx from GIT Repo - X
6. Start nginx server

E.g in kubernetes:

```
kubectl run --image=nginx nginx

kubectl edit deployment nginx

kubetl expose deployment nginx --port 80

kubectl scale deployment nginx --replicas=5

kubectl delete deployment nginx
```


### Declarative: Specify the final outcome, the system will then workout what to do and provision the environment

aka `Object Definition File`. Every kubectl apply command that works with yaml file is considered as **Declarative** command

E.g:

```yaml
VM NAME: Web-server
Database: nginx
Port: 8080
Path: /var/www/nginx
Code: GIT Repo - x
```

E.g in kubernetes

```
kubectl apply -f nginx.yaml
```





