![image](https://github.com/valkyron/kubernetes-improvement-scope/assets/45105530/8c014505-a51b-46b8-8df1-cc3153707ee6)
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

- Install WeaveNet CLI
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

Verify weavenet configuration:
```kubectl get pods -n kube-system```

- Setup 5 microservices with varying resource requirements:<br>
Create a separate YAML file for each microservice deployment. Each microservice will have different resource requirements specified.

- Make a docker image: ```docker build -t hello-world-node .```

- Deploy microservices:
```
kubectl apply -f microservice1-deployment.yaml
kubectl apply -f microservice1-service.yaml
# Repeat for other microservices
```

- Apply network policy:
 ```kubectl apply -f network-policy.yaml```

### Utilites

- Verifying clusters and nodes:

1. ```kind get clusters```
2. ```kubectl get nodes```
3. Check Weavernet pods status: ```kubectl get pods -n kube-system -l name=weave-net```

- How to stop all services

- ```
  kubectl delete deployment microservice1 microservice2 microservice3 microservice4 microservice5
kubectl delete service microservice1-service microservice2-service microservice3-service microservice4-service microservice5-service

kind delete cluster --name kind
  ```

- Verify cleanup
```
kubectl get all
docker ps
kind get clusters
```
