# k8s-notes
Kubernetes notes and commands

Exam Tips:

- https://afkham-azeez.medium.com/passing-ckad-tips-tricks-e24712f3e4a4
- https://kgamanji.medium.com/how-i-passed-my-ckad-with-97-6b54dcffa72f
- https://mengying-li.medium.com/failed-ckad-first-attempt-my-journey-to-improve-exam-score-from-53-to-98-in-a-month-part-1-badde0746231
- https://mengying-li.medium.com/failed-ckad-first-attempt-my-journey-to-improve-exam-score-from-53-to-98-in-a-month-part-2-f9e56f47cdf9
  - pay attention to provided hints
  - Do the context switch 
  - Use default namespace if not specificed
    - `alias kn='kubectl config set-context --current --namespace '
      kn default
  - Get familiar with nginx and busybox images.
  - Use `--restart=Never` for busybox image
    - `kubectl run busybox --image=busybox --command --restart=Never`
  - Use bookmarks because exam questions are similar to official examples
  - Use alias to avoid making mistakes
    - `alias kn='kubectl config set-context --current --namespace '`
  - Learn to validate the deployment is healthy by checking the log output and pods state
  - Learn to verify service healthiness by sending traffic to the service and check the response
- CKAD 
  - Udemy Course: https://www.udemy.com/course/certified-kubernetes-application-developer/
  - https://dzone.com/articles/kubernetes-for-application-developers-ckad
    - experience: https://github.com/IvoNet/CKAD-resources
- Cheatsheet:
  - https://github.com/dennyzhang/cheatsheet-kubernetes-A4/blob/master/cheatsheet-kubernetes-A4.pdf
- CKAD Exercises
  - https://github.com/dgkanatsios/CKAD-exercises
- prep notes
  - https://github.com/twajr/ckad-prep-notes#tasks-from-kubernetes-doc
- MCQ
  - https://testmoz.com/q/5560368
- Additional resources
  - https://github.com/bmuschko/ckad-crash-course#additional-resources
- https://www.youtube.com/watch?v=rnemKrveZks&ab_channel=MuralidaranShanmugham

## Know these linux commands
- curl
- wget 
- yq, jq
- 
## Setup
Pre Setup
- Autocompletion:
  - https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/

Once you've gained access to your terminal it might be wise to spend ~1 minute to setup your environment. You could set these:

````
alias k=kubectl                         # will already be pre-configured
export do="--dry-run=client -o yaml"    # k get pod x $do
export now="--force --grace-period 0"   # k delete pod x $now
````
- Vim
  - To make vim use 2 spaces for a tab edit ~/.vimrc to contain:
````
set tabstop=2
set expandtab
set shiftwidth=2
````

## General

- Run a command and exit
```bash

kubectl run tmp --restart=Never --rm -i  --image=nginx -- curl google.com

 ```
- check the client version
  - `kubectl version --client`
- View the config
  - `kubectl view config`
-- Get a list of all the k8s objects
- kubectl get all -A    

-- Get objects in namespace
kubectl get all -n namespace_name

-- getting help
- kubectl explain persistentvolume --recursive | less
- Cert tips
- attempt all questions 
- dont get stuck 
- Use shortcuts/ aliases
  - po for pods
  - rs for replicasets
  - deploy for deployments
  - svc for services
  - ns for namespaces
  - netpol for network policies
  - pv for persistent volume
  - pvc for persistent volume claim
  - sa for service account
- Additional tips

- find which node pods are running on 
  - kubectl get pods -o wide
- What does the READY column in the output of the kubectl get pods command indicate?
  - Running containers in the pod/ Total containers in the pod
  

### Exec in Container 
- ```kubectl exec pod-name -- cat filename ```

### Labels
- `kubectl get po --show-labels -l type=frontend`

### Get pods 
``` kubectl get pods ```

### ReplicaSets 
- replicaset ensures that desired number of pods are always run
- kubectl get rs
- kubectl get rs new-replica-set -o yaml | less
- kubectl edit rs new-replica-set
- kubectl scale replicaset
- kubectl edit replicaset
- Look up syntax
  - kubectl explain replicaset | grep VERSION
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: nginx
  template:
    metadata:
      labels:
        tier: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

### Deployments
- kubectl get deployments
- kubectl describe deployment name 
- kubectl edit deployment name 
  - Change image name
  - change deployment strategy 
- kubectl create -f deployment-definition-httpd.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      name: httpd-frontend
  template:
    metadata:
      labels:
        name: httpd-frontend
    spec:
      containers:
      - name: httpd-frontend
        image: httpd:2.4-alpine

```

### ConfigMap 
- kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
 ```yaml 
  spec:
  containers:
  - envFrom:
    - configMapRef:
      name: webapp-config-map
  ```
  
### Secret
- kubectl create secret generic db-secret --from-literal=DB_Host=sql-1 --from-literal=DB_User=root --from-literal=DB_Password=password123
```yaml
spec:
  containers:
    - image: kodekloud/simple-webapp-mysql
      envFrom:
        - secretRef:
            name: db-secret
```

### namespaces
- How many pods exist in the research namespace
  - kubectl get pods -n research  
- kubectl run redis --image=redis -n finance
  - kubectl run redis --image=redis -n finance
- Which namespace has the pod blue 
  - kubectl get pods --all-namespaces | grep blue
### imperative commands 
- Create pod with the image given
  - kubectl run podtest --image=nginx
  - kubectl run nginx-pod --image nginx:alpine
- Create pod and add a label 
  - kubectl run redis --image redis:alpine
  - kubectl label pod redis tier=db
- Create a service redis-service to expose application within the cluster on the port 6379
  - kubectl expose pod redis --port=6379 --name=redis-service
- Create Deployment
  - kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
- Create a new pod called custom-nginx using the nginx image and expose it on container port 8080
  - kubectl run custom-nginx --image=nginx --port=8080
- Create a new name namespace
  - kubectl create ns dev-ns
- Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas.
  - kubectl create deployment redis-deploy -n dev-ns --image=redis --replicas=2
- Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80.
- ```yaml 
  root@controlplane:~# kubectl run httpd --image httpd:alpine --port 80 --expose
  service/httpd created
  pod/httpd created
  ```
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
- service acts as a built-in load balancer
- cluster ip service type
        - targetPort: 80
        - port: 80
- service layer can assign port number greather that 30000
- Describe service in a namespace 
  - kubectl describe service db-service -n dev
  

#### commands
- Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

- ```kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml```
- **(This will automatically use the pod’s labels as selectors)**

- Or

- ```kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml```
- **This will not use the pods labels as selectors**, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. 
- So generate the file and modify the selectors before creating the service


- Create a Service named nginx of type NodePort to expose pod nginx’s port 80 on port 30080 on the nodes:
  - ```kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml```
    - (This will automatically use the pod’s labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

- ```kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml```

- **(This will not use the pods labels as selectors)**

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port.
- KodeCloud recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.




        - kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
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

- check role binding

  - kubectl -n ingress-space get roles.rbac.authorization.k8s.io
  - kubectl -n ingress-space get rolebindings.rbac.authorization.k8s.io

- Mount volume
- kubectl run webapp --image=kodekloud/event-simulator --dry-run=client -o yaml > o.yaml
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: webapp
  name: webapp
spec:
  volumes:
          - name: log-volume
            hostPath:
              path:  /var/log/webapp
              type: Directory
  containers:
  - image: kodekloud/event-simulator
    name: webapp
    volumeMounts:
            - mountPath: /log
              name: log-volume
```

- Create Persistent Volume
```

apiVersion: v1
kind: PersistentVolume
metadata:
        name: pv-log
spec:
        persistentVolumeReclaimPolicy: Retain
        accessModes:
                - ReadWriteMany
        capacity:
                storage: 100Mi
        hostPath:
                path: /pv/log
~                                                                                                                             

```

- Create Persistent Volume Claim 
- kubectl describe persistentVolumeClaim claim-log-1
- kubectl describe pvc claim-log-1
- kubectl delete pvc claim-log-1

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
        name: claim-log-1
spec:
        accessModes:
                - ReadWriteMany
        resources:
                 requests:
                         storage: 50Mi

```

- Docker Images
  - How many docker images are present? `docker images` & count :)
  - Build an image from Dockerfile `docker build -t image-name .`
  - Run image `docker run  -p host-port:container-port image-name` e.g. `docker run -p 8282:8080 webapp-color`
  - To run the container in the background, add the `-d` flag.

- KubeConfig
  - `kubectl config use-context research --kubeconfig /root/my-kube-config`

- Inspect the env
  - `kubectl describe pod kube-apiserver-controlplane -n kube-system`
- # of roles
  - `kubectl get roles --all-namespaces | wc --l`
- 
- What access is given to role Kube-proxy in kube system namesystem
  - `kubectl describe role kube-proxy -n kube-system`

- RoleBinding
  - `kubectl describe rolebinding kube-proxy -n kube-system`

- Check access
  - Inspect the permissions granted to the user. Check if the user can list pods in the default namespace.
  - `kubectl get pods --as dev-user`

- Create Role Binding
  - `kubectl create rolebinding dev-user-binding --clusterrole=developer --user=dev-user`
- Edit Role
  - `kubectl edit role developer -n=blue`

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: blue
  name: deploy-role
rules:
- apiGroups: ["apps", "extensions"]
  resources: ["deployments"]
  verbs: ["create"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-deploy-binding
  namespace: blue
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: deploy-role
  apiGroup: rbac.authorization.k8s.io

```

- Admission controllers
  - grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml
  - Add NamespaceAutoProvision admission controller to --enable-admission-plugins list to /etc/kubernetes/manifests/kube-apiserver.yaml 
  - It should look like below
    - `--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision`
    - API server will automatically restart and pickup this configuration
  - To disable a plugin
    - Update `/etc/kubernetes/manifests/kube-apiserver.yaml` as below
    - `--disable-admission-plugins=DefaultStorageClass`
  - Checkout enabled and disabled plugins
    - Since the kube-apiserver is running as pod you can check the process to see enabled and disabled plugins.
    - `ps -ef | grep kube-apiserver | grep admission-plugins`

  - Create TLS secret
    - `root@controlplane:~# kubectl -n webhook-demo create secret tls webhook-server-tls --cert "/root/keys/webhook-server-tls.crt" --key "/root/keys/webhook-server-tls.key"
      secret/webhook-server-tls created`
    - Webhook config
  ```yaml
  
    apiVersion: admissionregistration.k8s.io/v1beta1
    kind: MutatingWebhookConfiguration
    metadata:
      name: demo-webhook
    webhooks:
     - name: webhook-server.webhook-demo.svc
       clientConfig:
       service:
          name: webhook-server
          namespace: webhook-demo
          path: "/mutate"
          caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURQekNDQWllZ0F3SUJBZ0lVR2tlYXNhS25yNkhkRkE4NERDZEIwaTRCWFJrd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0x6RXRNQ3NHQTFVRUF3d2tRV1J0YVhOemFXOXVJRU52Ym5SeWIyeHNaWElnVjJWaWFHOXZheUJFWlcxdgpJRU5CTUI0WERUSXhNVEV3TVRFek1USXdNMW9YRFRJeE1USXdNVEV6TVRJd00xb3dMekV0TUNzR0ExVUVBd3drClFXUnRhWE56YVc5dUlFTnZiblJ5YjJ4c1pYSWdWMlZpYUc5dmF5QkVaVzF2SUVOQk1JSUJJakFOQmdrcWhraUcKOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXBIRzFyQkdQdkNnME8vb0ZJRWVSM0JsUURDR2pHOWZySm9pRQpGSG1EVzBuNGx2MGNNTVd0dDVQVmkzYmlZRkc5RmwyZVZTTmk5S2JtWmJpNUVxNTdOeDBITWZCS0Zldi9hRDdmClkzSkFMenptblh1SytGTXpkM2d0dU00cnNuOHJrMXR2QW8rWVNqNE1aLzc4TkkvZG9tYnlROUMzdlZZODVoYVkKdkFWU0kzQy9Zd1FFdjRtVDVNV0Nqb3MvalMxSmhHVHNvU3dIMWtaK0V3VjluMG9PWHVaN1VQMWQrWUMvTUVTVgpockNOTldsNE9wNzNldENtS012a2JUTGZDM0Rvc0tHNnZiNGxhNDdsU1hva05ESEpHMmd1L0tnN3Q3S0VKc3hMCndKS3JONDFXYlNIS3M2Uk1TNnpSaG1PakNGRGw4MU5KeTFGc21MRXFNMmwraC9QSXdRSURBUUFCbzFNd1VUQWQKQmdOVkhRNEVGZ1FVU2UzV3VNU2pOZjJRa1IvU3NXRlZodk5EM0Frd0h3WURWUjBqQkJnd0ZvQVVTZTNXdU1TagpOZjJRa1IvU3NXRlZodk5EM0Frd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DCkFRRUFhOXFzM2FPb0drNnZTWUM0NERhcExkK0FETWVCbUw1MkUyMzEvbGVTZ1daRklqdm5kTmlCb21FeUZZZG4KTU54em14MTh0SGxMK1JaYkFUcjAzQlM3bURyNUVpbi9INHRSSmhzY0xoUWIxZkQwNVNianIvVmNwL0RQR09obApwQkJYcHdBcTl0RFNMOFQxWGYxaWxQUDJraXcwV2pxSk84SVI1SWJZbzNxT0gvbUlqVThxekFTYmtMWTFrcVRjCngra0wvT1VFTXpFVDhIdEFINUJHU2N1L25UNGxWN0orbjlWRUh2czMyVHF5Y1Y4Sm5oSHdRM3ZSMTgvOWJpRFgKWWIzRVFreWUzR1RKTGpNSi9ZUU1NUktKOVM0MXV5YlRaTWJqa3Z0S1JoRk4yTW45dExURDQrTjdFWFZ1WG8wcwpGa0JUWjg1UFpqMmZydmlQN3RldjUwN2FvQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    rules:
     - operations: [ "CREATE" ]
       apiGroups: [""]
       apiVersions: ["v1"]
       resources: ["pods"]
    
   ```

- Identify the short names of the deployments, replicasets, cronjobs and customresourcedefinitions.
  - `kubectl get rs,deploy,cj,crd -A`
 
- find the job family or api group
  - `kubectl explain job`

- To identify the preferred version, run the following commands as follows :-
  - `root@controlplane:~# kubectl proxy 8001&`
  - `root@controlplane:~# curl localhost:8001/apis/authorization.k8s.io`
- Where & runs the command in the background and kubectl proxy command starts the proxy to the kubernetes API server.


- Enable the v1alpha1 version for rbac.authorization.k8s.io API group on the controlplane node.
  - Add the --runtime-config=rbac.authorization.k8s.io/v1alpha1 option to the kube-apiserver.yaml file.
- As a good practice, take a backup of that apiserver manifest file before going to make any changes.
  In case, if anything happens due to misconfiguration you can replace it with the backup file.
```
root@controlplane:~# cp -v /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.backup
Now, open up the kube-apiserver manifest file in the editor of your choice. It could be vim or nano.

root@controlplane:~# vi /etc/kubernetes/manifests/kube-apiserver.yaml
Add the --runtime-config flag in the command field as follows :-

- command:
  - kube-apiserver
  - --advertise-address=10.18.17.8
  - --allow-privileged=true
  - --authorization-mode=Node,RBAC
  - --client-ca-file=/etc/kubernetes/pki/ca.crt
  - --enable-admission-plugins=NodeRestriction
  - --enable-bootstrap-token-auth=true
  - --runtime-config=rbac.authorization.k8s.io/v1alpha1 --> This one
    After that kubelet will detect the new changes and will recreate the apiserver pod.

It may take some time.

root@controlplane:~# kubectl get po -n kube-system
Check the status of the apiserver pod. It should be in running condition.
```
- Change version using kubectl-convert
  - `kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1`
  - Make the change and save it to file
    - `kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml`
    
Deployment
```yaml
 kubectl rollout history deployment/api-new-c32 -n=neptune --revision=4
 kubectl rollout undo deployment/api-new-c32 -n=neptune
```
