## Daemonsets

```
k get daemonset kube-proxy -n kube-system -o yaml
kubectl describe daemonset kube-proxy --namespace=kube-system


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
