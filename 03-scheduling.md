# nodeName

```yaml
...
spec:
  # to schedule pod onto node without scheduler use nodeName
  nodeName: node01
```

# Labels and Selectors

Used to group and select objects

`kubectl get pods -l env=prod,tier=frontend`

# Taint and Toleration

Taint and toleration is node selecting pods that can be scheduled onto them, where as nodeAffinity is pods selecting the nodes they want to schedule on 

### Taint 

Taint are for nodes are for nodes

`kubectl taint nodes node-name key=value:taint-effect`

E.g:

`kubectl taint nodes node1 app=blue:NoSchedule`

3 taint effects

* NoSchedule - pods will not be scheduled on node
* PreferNoSchedule - pods will likey not to be scheduled on node, no guarantee 
* NoExecute - new pods wont get scheduled, existing pods will be evicted if pod doesnt tolerate taint

**To remove taints:**

Add minus sign at the end of taint command

`kubectl taint nodes node1 app=blue:NoSchedule-`


### Toleration


Tolerations are for pods, see how each item in a toleration matches up with a taint from `kubectl taint nodes node1 app=blue:NoSchedule`

```yaml
spec:
  ...
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```


# nodeSelector

To label a node:

`kubectl label nodes node_name key=value`

`kubectl label nodes node01 size=large`


Pod definition to schedule pod to node with label: size=Large

```yaml
spec: 
  ...
  nodeSelector:
    size: Large
```
 
 
# nodeAffinity

```yaml
spec:
  ...
  affinity:
    nodeAffinity:
      # requiredDuringSchedulingIgnoredDuringExecution
      # preferredDuringSchedulingIgnoredDuringExecution
      # requiredDuringSchedulingRequiredDuringExecution
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            # In, NotIn option, Exists
            operator: In
            values:
            - Large 
            - Medium
```
 
**requiredDuringSchedulingIgnoredDuringExecution:**

Strict rule, kubernetes won't schedule pod to any node even if no node matches requirement, current pod stays even if doesnt match

**preferredDuringSchedulingIgnoredDuringExecution:**

Chill rule, kubernetes will schedule pod to node after verifying there's no available node to schedule on

**requiredDuringSchedulingRequiredDuringExecution**

Super strict rule, kubernetes won't schedule pod to any node even if no node matches requirement, current pod evicted if doesnt match 

    
# Resource Requirements and Limits

* kubernetes default 1cpu limit & 512Mi mem for pods

* pod going over cpu limit will get throttled

* pod going over mem limit will get terminated


```yaml
...
spec:
  resources:
    requests:
      memory: "1Gi"
      cpu: 1
    limits:
      memory: "2Gi"
      cpu: 2  
```

one can specify LimitRange for namespace. E.g: 

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

# DaemonSets

* Run one copy of each pod on each node, whenever a node comes up that pod run in the new node.
* Use case: `kube-proxy, networking (calico,weavenet..etc)`
* daemonset use nodeaffinity to schedule pods onto node


```yaml
apiVersion: apps/v1
Kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  # same as delpoyment, replicaset
  ...  
``` 

# Static pods

* one can create pod on kubelet without kubernetes master / api-server
* you can inspect definition file in kubelet.service under `--config=kubeconfig.yaml`
* use `docker ps` to get running containers in host
* kubernetes master is aware of static pods because kubelet creates a mirror object in kubernetes api, 
but you can only modify static pod in node and not via kubernetes api
* get static pod definition file location by:

```
Run the command ps -aux | grep kubelet and identify the config file - --config=/var/lib/kubelet/config.yaml. 
Then checkin the config file for staticPdPath.
```

### difference between static pods & daemonsets

static pods:

* created by kubelet
* used to deploy control-plan components
* ignored by kube-scheduler

daemonsets:

* created by kubernetes api server
* ignored by kube-scheduler


# Multiple Scheduler

kubeadm deploy kube-scheduler in yaml file `/etc/kubernetes/manifests/kube-scheduler.yaml`

One can deploy another scheduler just by copying the file above and change name as well as set `--scheduler-name=custom-scheduler`, 
you can also set a `--lock-object-name=customer-scheduler` to differentiate this scheduler with another scheduler during the leader election process


Use `kubectl get events` to find out what scheduler was used to schedule a pod, under SOURCE column. View logs of scheduler in scheduler pod

### Example - create additional scheduler

create following file in `/etc/kubernetes/manifests`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: my-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --port=30051
    - --leader-elect=false
    - --scheduler-name=my-scheduler
    image: k8s.gcr.io/kube-scheduler:v1.18.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
```

* `- --port=30051` to specify port
* `- --leader-elect=false` to stop leader election
* `- --scheduler-name=my-scheduler` to name scheduler

### Example - use new scheduler for pod 


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotation-default-scheduler
spec:
  schedulerName: my-scheduler
  containers:
  - name: pod-with-default-annotation-container
    image: k8s.gcr.io/pause:2.0
```
