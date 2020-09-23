# Rollout

Get current status of rollout for deployment 

`kubectl rollout status deployment/myapp`

Get revision/history of rollout for deployment

`kubectl rollout history deployment/myapp`

Rollback to previous version

`kubectl rollout undo deployment/myapp`

# Deployment strategy

* Recreate - takes down all current and schedule new pods
* RollingUpdate - take a portion (default to 25%) of pods down and schedule in new pods, 
keep rolling till all pods are replaced by new pods

# Deployment lifecycle

update deployment -> create new replicaset -> old replicaset set to 0 pods

rollback deployment -> old replicaset with running pods -> new replicaset set to 0 pods

# Docker CMD & ENTRYPOINT

### CMD

CMD in docker run needs to be separated by comma. E.g - `CMD["sleep", "5"]` instead of `CMD["sleep 5"]`

E.g - A container `ubuntu-sleeper` with `ENTRYPOINT ["sleep"]` and a run command of `docker run ubuntu-sleeper 10` will fail because 10 is not a valid command

### ENTRYPOINT

ENTRYPOINT in docker container specify the command to be ran on container start. 

E.g - A container `ubuntu-sleeper` with `ENTRYPOINT ["sleep"]` and a run command of `docker run ubuntu-sleeper 10` will sleep for 10s

In other words, CMD will **overwrite** run command in `docker run ubuntu-sleep 10` to `10`. Whereas ENTRYPOINT will **append** 10 to ENTRYPOINT command `sleep`


You can specify CMD with ENTRYPOINT together to form a dockerfile that looks like this:

```
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

* this container will run sleep 5 at startup
* `docker run ubuntu-sleeper 10` would still work because ENTRYPOINT is set to `sleep`
* you can modify entrypoint at runtime with `docker run --entrypoint sleep2.0 ubuntu-sleeper 10`

# Commands and Arguments in Pod

```yaml
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      # command overwrite ENTRYPOINT in container
      command: ["sleep"]
      # args overwrite CMD in container
      args: ["10"]
```

set argument `--colour=green` for command `set-colour`

```yaml
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      # command overwrite ENTRYPOINT in container
      command: ["set-colour"]
      # args overwrite CMD in container
      args: ["colour", "green"]
```

# Set ENV Variables in Pod

```yaml
spec:
  containers:
    - name: ubuntu-sleeper
    ...
    env:
      # set env variable directory
      - name: COLOUR
        value: BLUE
      # set env var via configmap
      - name: COLOUR
        valueFrom:
          configMapKeyRef:   
      - name: COLOUR
        valueFrom:
          secretKeyRef:
```

# ConfigMaps

Create configmap via command line

`kubectl create configmap app-config --from-literaly=APP_COLOR=blue --from-literaly=APP_ENV=prod`

Create configmap from file, data is stored under name of file in configMap.

`kubectl create configmap app-config --from-file=<file-path>`

Create configmap via yaml, `kubectl apply -f file.yml` 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

Read configmap data with: 

`kubectl describe configmap <name>`

#### Use configmap in pods

configmap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

pod to use configmap `app-config` created above

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: ubuntu
    ports:
      - containerPort: 8080
    envFrom:
      - configMapRef:
          name: app-config
```

# Secrets

create secret via command line

`kubectl create secret generic app-secret --fron-literal=key=value --fron-literal=key=value`

Create secret from file, data is stored under name of file in secret

`kubectl create secret app-secret --from-file=<file-path>`

Create configmap via yaml, `kubectl apply -f file.yml` , value needs to be passed as base64 encoded string. 
Using command such as `echo -n 'string' | base64`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  # value needs to be base64 encoded, showing true value for teaching purposes
  DB_Host: mysql
  DB_User: root
  DB_password: paswrd
```

Some notes on secret:

* A secret is only sent to a node if a pod on that node requires it.

* Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.

* Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.



#### Use secret in pods as environment variables

secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw
  DB_User: cm9vdA==
  DB_password: cGFzd3Jk
```

pod to use secret `app-secret` created above

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: ubuntu
    ports:
      - containerPort: 8080
    envFrom:
      - secretRef:
          name: app-secret
```

#### Use secret in pods as volume

secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw
  DB_User: cm9vdA==
  DB_password: cGFzd3Jk
```

pod to use secret `app-secret` created above

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: ubuntu
    ports:
      - containerPort: 8080
  volumes:
  - name: app-secret
    secret:
      secretName: app-secret  
```

injecting secret as volume will create each item as a file in the volume path, so within the pod you'd find these files:

* /opt/app-secret-volume/DB_Host
* /opt/app-secret-volume/DB_User
* /opt/app-secret-volume/DB_password

and if you cat the file `/opt/app-secret-volume/DB_password` you'd see value `paswrd` in it, that's already based64 decoded by kubernetes

# InitContainers

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion 
before the real container hosting the application starts. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
```