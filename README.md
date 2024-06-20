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

- Make a docker image: ```docker build -t hello-world-node:latest .```

- Deploy microservices:
```
kubectl apply -f microservice1-deployment.yaml
kubectl apply -f microservice1-service.yaml
# Repeat for other microservices
```

- Check if the microservices are working: ```kubectl get pods -o wide```

![image](https://github.com/valkyron/kubernetes-improvement-scope/assets/45105530/6a7ff172-33ff-41ba-9e77-fcd3e843d372)

- Apply network policy:
 ```kubectl apply -f network-policy.yaml```

- Check if the network policy is applied: ```kubectl get networkpolicy``` 
- Describe the NetworkPolicy to verify its details: ```kubectl describe networkpolicy allow-specific```

### Analyzing

- Print kube scheduler logs:
```
kubectl logs -n kube-system kube-scheduler-kind-control-plane
```

- Print events related to pods in the 'kube-system' namespace
```kubectl get events --namespace kube-system --field-selector involvedObject.kind=Pod```

## Improvements

1. Pod priority is not considered and a randomly selected pod is scheduled on the node with the highest node score, and hence the global optimal solution cannot be made <br>
**Solution:** Queue Sort Plugin

2. The containers can be be distributed across different nodes - this can be done using **Node Affinity**
```
      spec:
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: kubernetes.io/hostname
                  operator: In
                  values:
                  - <node-name>  # Replace with your node name
```

and apply on that microservice: ```kubectl apply -f microservice-1.yaml``` <br>
**Solution:** Split and Distribute Extender Plugin

3. Only CPU and RAM usage rates are considered in the service scheduling while latency or bandwidth usage rates are not considered at all. <br>
**Solution:** Topology Aware Scoring Extender Plugin

### Utilites

- Verifying clusters and nodes:

1. ```kind get clusters```
2. ```kubectl get nodes```
3. Check Weavernet pods status: ```kubectl get pods -n kube-system -l name=weave-net```
4. Check all pods are running: ```kubectl get pods -o wide```
5. Check images on each node:
```
docker exec -it kind-control-plane crictl images | grep hello-world-node
docker exec -it kind-worker crictl images | grep hello-world-node
docker exec -it kind-worker2 crictl images | grep hello-world-node
docker exec -it kind-worker3 crictl images | grep hello-world-node
docker exec -it kind-worker4 crictl images | grep hello-world-node
docker exec -it kind-worker5 crictl images | grep hello-world-node
```

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
