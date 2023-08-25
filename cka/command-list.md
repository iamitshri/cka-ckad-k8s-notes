### NEtworking 

<img width="1127" alt="image" src="https://github.com/iamitshri/cka-ckad-k8s-notes/assets/26192212/b88ef0fb-ab1a-4645-bfd0-1c4ee26806b1">

#### kubectx

```
While this is excellent for hands-on practice, in a real “live” kubernetes cluster implemented for production, there could be a possibility of often switching between a large number of namespaces and clusters.

This can quickly become and confusing and overwhelming task if you had to rely on kubectl alone.

This is where command line tools such as kubectx and kubens come in to picture.

Reference: https://github.com/ahmetb/kubectx

Kubectx:

With this tool, you don’t have to make use of lengthy “kubectl config” commands to switch between contexts. This tool is particularly useful to switch context between clusters in a multi-cluster environment.

```

### netpol
```

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
      matchLabels:
          name: internal

  policyTypes:
    - Egress


  egress:
      - to:
        - podSelector:
            matchLabels:
              name: payroll
        ports:
        - port: 8080
          protocol: TCP
      - to:
        - podSelector:
            matchLabels:
              name: mysql
        ports:
        - port: 3306
          protocol: TCP

```

### ClusterRole and Role
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: storage-admin
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "watch", "list", "create", "delete"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-storage-admin
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: storage-admin
  apiGroup: rbac.authorization.k8s.io


k auth can-i get pods/dark-blue-app  --as dev-user -n blue
k auth can-i get pod --as dev-user
k create rolebinding dev-user-binding --role=developer --user=dev-user
k create role developer --verb=delete,list,create --resource=pods

```
<img width="720" alt="image" src="https://github.com/iamitshri/cka-ckad-k8s-notes/assets/26192212/f6c59810-0013-47ee-bb25-ecfab30cef7f">

```
etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=https://127.0.0.1:2379  snapshot save /opt/snapshot-pre-boot.dbSnapshot saved at /opt/snapshot-pre-boot.db

```

### find etcd server url
Use the command kubectl describe pod etcd-controlplane -n kube-system and look for --listen-client-urls
can also use ps -aux | grep etcd and find the process kube-apiserver 

### Draining node does not work on pod not part of replicaset 
`k drain node01 --ignore-deamonsets`
If pod is not part of a replicaset. The drain command will not work in this case. To forcefully drain the node we now have to use the --force flag.

### note on self healing apps

Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.
Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes.



### using custom col
```
k get po -o custom-columns=c:.metadata.name,IMAGE:.spec.initContainers[].image,state:.status.initContainerStatuses[0].state
```
### Look at log in multicontainer pod 
```
k -n elastic-stack exec -it app -- cat /log/app.log

```
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
