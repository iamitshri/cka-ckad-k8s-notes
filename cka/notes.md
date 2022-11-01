### etcd Database
- Key value store
- Stores everything about cluster: nodes, pods, configs, secret, accounts, roles, bindings, other
- Default port 2379


### Kube API Server
- authenticates, validates request, retrieve/update request data to/from etcd

### Kube Controller Manager
- Controller monitors  continuously state of various components and work towards bringing them to the desired 
