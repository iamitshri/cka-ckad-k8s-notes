# k8s-notes
Kubernetes notes and commands

## General

-- Get a list of all the k8s objects
- kubectl get all -A    

-- Get objects in namespace
kubectl get all -n namespace_name

### Get pods 
``` kubectl get pods ```

### Deployments
- kubectl get deployments
- kubectl describe deployment name 
- kubectl edit deployment name 
  - Change image name
  - change deployment strategy 

### Logs
- kubectl logs pod-name

### Jobs
- kubectl get jobs
- kubectl get cronjob
- kubectl delete job throw-dice-job

```
apiVersion: batch/v1
kind: Job
metadata:
        name: throw-dice-job
spec:
        backoffLimit: 25
        completions: 3
        parallelism: 3
        template:
                spec:
                         containers:
                                 - image: kodekloud/throw-dice
                                   name: throw-dice-job
                         restartPolicy: Never
 ```
 - Creating Cron Job
 ```
 apiVersion: batch/v1beta1
kind: CronJob
metadata:
        name: throw-dice-job
spec:
        schedule: "30 21 * * *"
        jobTemplate:
                spec:
                         backoffLimit: 25
                         completions: 3
                         parallelism: 3
                         template:
                                spec:
                                  containers:
                                      - image: kodekloud/throw-dice
                                        name: throw-dice-job
                                  restartPolicy: Never
 ```


### Services
- Services enables communication between group of pods
- promotes loose coupling between pods
- listens on port on node and forward traffic to pod
- Nodeports Service type
  - service port aka targetPort
  - nodePort
  - port 
 - service acts as a built in load balancer
- cluster ip service type
        - targetPort: 80
        - port: 80
- service layer can assign port number greather that 30000
#### commands
        - kubectl get services
        - kubectl describe service kubernetes
        - kubectl edit service kubernetes
        - kubectl get endpoints
        - kubectl expose deployment name --name=svc-name --target-port=8080 --port=8080  --node-port=30080 --type=Nodeport --dry-run=client -o yaml > svc.yaml

- Ingress
- kubectl get ingress --all-namespaces
- kubectl describe ingress --namespace app-space
- kubectl edit ingress --namespace app-space
- kubectl get deployments -n app-space
- kubectl get svc -n critical-space

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
           name: pay-service
           port:
            number: 8282


```

kubectl get deployments.apps --all-namespaces
kubectl get ingress --all-namespaces
kubectl describe ingress 
kubectl describe -n app-space ingress name
kubectl get deployments.apps,svc 
kubectl get deployments.apps --all-namespaces

kubectl create ns ingress-space
kubectl create configmap nginx-configuration --namespace ingress-space
kubectl create serviceaccount ingress-serviceaccount --namespace ingress-space

kubectl get serviceaccounts -n ingress-space

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-controller
  namespace: ingress-space
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --default-backend-service=app-space/default-http-backend
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443

```

- Create service
kubectl expose deployment ingress-controller --type=NodePort --port=80 --name=ingress --dry-run=client -o yaml > ingress.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress-space
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30080
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress

```

- Create ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
        name: ingress-wear-watch
        namespace: app-space
        annotations:
                nginx.ingress.kubernetes.io/rewrite-target: /
                nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
        rules:
                - http:
                        paths:
                                - path: /wear
                                  pathType: Prefix
                                  backend:
                                          service:
                                                  name: wear-service
                                                  port:
                                                          number: 8080
                                - path: /watch
                                  pathType: Prefix
                                  backend:
                                          service:
                                                  name: video-service
                                                  port:
                                                          number: 8080

```

kubectl -n ingress-space get roles.rbac.authorization.k8s.io
kubectl -n ingress-space get rolebindings.rbac.authorization.k8s.io