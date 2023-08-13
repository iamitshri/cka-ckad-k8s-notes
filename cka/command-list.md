# Taints and Tolerations
```
k taint node node01 spray=mortein:NoSchedule 
k run mosquito --image nginx
k get po mosquito -o jsonpath='{.status.phase}'
# check taint on node
k describe node node01 | grep -i taint
#remove taint
k taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
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
