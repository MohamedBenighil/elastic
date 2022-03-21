## Overview
This doc creates elasticsearch cluster having node roles. The process is done using k8s operators.

## pre-requisists
Kubernetes cluster

## Steps

### Install CRD
`kubectl create -f https://download.elastic.co/downloads/eck/2.1.0/crds.yaml`

### Install Operators with its RBAC rules
`kubectl apply -f https://download.elastic.co/downloads/eck/2.1.0/operator.yaml`


### Deploy manifest
```
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 8.1.0
  nodeSets:
  - name: masters
    count: 1
    config:
      node.roles: ["master"]
      xpack.ml.enabled: true
    podTemplate:
      spec:
        initContainers:
        - name: sysctl
          securityContext:
            privileged: true
            runAsUser: 0
          command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
        volumes:
        - name: elasticsearch-data
          emptyDir: {}
  - name: data
    count: 1
    config:
      node.roles: ["data"]
    podTemplate:
      spec:
        initContainers:
        - name: sysctl
          securityContext:
            privileged: true
            runAsUser: 0
          command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
        volumes:
        - name: elasticsearch-data
          emptyDir: {}
  - name: hot
    count: 1
    config:
      node.roles: ["data_hot"]
    podTemplate:
      spec:
        initContainers:
        - name: sysctl
          securityContext:
            privileged: true
            runAsUser: 0
          command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
        volumes:
        - name: elasticsearch-data
          emptyDir: {}
```

- There is no attached Volumes.
- There is init container to set `vm.max_map_count=262144` on nodes.
