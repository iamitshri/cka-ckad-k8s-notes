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

- Info from Killer.sh
Minimal Setup
Alias

The alias k for kubectl will be configured together with autocompletion. In case not you can configure it using this link.

### Vim

Create the file ~/.vimrc with the following content:

```
set tabstop=2
set expandtab
set shiftwidth=2
```
The expandtab make sure to use spaces for tabs. Memorize these and just type them down. You can't have any written notes with commands on your desktop etc.

Optional Setup
### Fast dry-run output

`export do="--dry-run=client -o yaml"`
This way you can just run k run pod1 --image=nginx $do. Short for "dry output", but use whatever name you like.

### Fast pod delete

`export now="--force --grace-period 0"`
This way you can run k delete pod1 $now and don't have to wait for ~30 seconds termination time.

Persist bash settings

You can store aliases and other setup in ~/.bashrc if you're planning on using different shells or tmux.

### Alias Namespace

In addition you could define an alias like:

`alias kn='kubectl config set-context --current --namespace` Which allows you to define the default namespace of the current context. Then once you switch a context or namespace you can just run:

```yaml
kn default        # set default to default
kn my-namespace   # set default to my-namespace
```

But only do this if you used it before and are comfortable doing so. Else you need to specify the namespace for every call, which is also fine:
```
k -n my-namespace get all
k -n my-namespace get pod
```
 

- Get the current namsepace
- `kubectl config view --minify --output 'jsonpath={..namespace}{"\n"}'; `