# kubernetes-improvement-scope
Implementing a research paper to improve upon the existing kubernetes use case using an example kubernetes 5-node cluster
System: Windows

## Prerequisites

- WSL 2, Docker
- Kubectl:
```
 curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.7.0/bin/windows/amd64/kubectl.exe
```

- Kind: to easily set up multi-node clusters locally (Install go as well)
```
go install sigs.k8s.io/kind@v0.23.0
```

## Create the cluster:

- touch cluster-config.yaml
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
- role: worker
- role: worker
```

- Setup cluster:
```
kind create cluster --config=cluster-config.yaml
```
Setup 
