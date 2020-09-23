# Exam Tips

As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. 
During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. 

Using the kubectl run command can help in generating a YAML template. 


For example, if you were asked to create a pod or deployment with specific name and image you can simply run the kubectl run command.

`kubectl run --generator=run-pod/v1 nginx --image=nginx`


`kubectl run redis --image=redis --dry-run -oyaml > pod.yml` 


### Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

`kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml`


### Run pod with command

`kubectl run busybox --image busybox --comand -- sleep 1000`



### Create a deployment

`kubectl create deployment --image=nginx nginx`


### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

`kubectl create deployment --image=nginx nginx --dry-run -o yaml`

### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

`kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml`

Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.


### Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

### Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

`kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml`


