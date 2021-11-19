## Q & A 

<details><summary>expander</summary>
<p>

````bash

test this
````
</p>


</details>

- The DevOps team would like to get the list of all Namespaces in the cluster. Get the list and save it to /opt/course/1/namespaces.
   - ```k get ns -A > /opt/course/1/namespaces```

```bash
number yy : copy number of lines
 e.g. 1yy, 2yy will copy 2 lines

 number dd :- delete line
 e.g. 2dd delete 2 lines

 number p : paste lines below the current line

 x delete character 

number undo or number u : undo changes 

redo 

gg: go to start of line 

G go to end of file 
 ```


 - yq
 - jq
 - curl
 - wget

 - Shortcuts for speed
 - Add these commands in `~/.bashrc`
```bash
alias k="kubectl"
alias k="k "
alias kg="k get"
alias kd="k describe"
alias kr="k run"
alias kcr="k create"
alias ka="k apply -f"
alias krm="k delete"

# usage: kn <namespace>
alias kn="k config set-context --current --namespace"

export dry="--dry-run=client -o yaml"

#example
kr $dry web --image=nginx:alpine

# generating yaml
kg po web -o yaml

# Generating a deployment YAML
kcr $dry deploy web --image=nginx:alpine --replicas=3

# Generating Service
k expose $dry deploy web --port=8080 --target-port=80
```
