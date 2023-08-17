### Setting env using secretRef

```
  containers:
  - image: kodekloud/simple-webapp-mysql
    envFrom:
     - secretRef:
        name: db-secret

```

### Setting env using configmap
```
  - env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          key: APP_COLOR
          name: webapp-config-map
```

### Deployment 

```

k set image  deploy/frontend simple-webapp=kodekloud/webapp-color:v3

```

### Static pods
- pods ending with node name
- created by kubelet without the help of api server
- create pod defs in the configured location to create static pod /etc/kubernetes/manifests 
```
k get pod -A | egrep  -i '\-controlplane'

# Run the command ps -aux | grep kubelet and identify the config file - --config=/var/lib/kubelet/config.yaml. Then check in the config file for staticPodPath.
# see the option --config=/var/lib/kubelet/config.yaml
# open kubelet config and checkout value of staticPodPath


kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml

```
```yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - command:
    - sleep
    - "1000"
    image: busybox:1.28.4
    name: static-busybox
```



## Daemonsets

```
k get daemonset kube-proxy -n kube-system -o yaml
kubectl describe daemonset kube-proxy --namespace=kube-system

# Create Deployment and edit it by removing replica, strategy update kind
apiVersion: apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  labels:
    app: elasticsearch
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - image: registry.k8s.io/fluentd-elasticsearch:1.20
        name: fluentd-elasticsearch

```

## Node Affinity

```
# create deployment
k create deployment blue --image nginx --replicas 3

    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: node-role.kubernetes.io/control-plane
                  operator: Exists
```

```
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: color
                  operator: In
                  values:
                    - blue
```

## Node Selector 

```
nodeSelector:
    labelName: value

```

## Taints and Tolerations
```
k taint node node01 spray=mortein:NoSchedule 
k run mosquito --image nginx
k get po mosquito -o jsonpath='{.status.phase}'
# check taint on node
k describe node node01 | grep -i taint
#remove taint
k taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
k describe node controlplane | grep -C 10 -i taints
```

### pod with tolerations 
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: bee
  name: bee
spec:
  tolerations:
    - key: spray
      value: mortein
      operator: Equal
      effect: NoSchedule
  containers:
  - image: nginx
    name: bee
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```
