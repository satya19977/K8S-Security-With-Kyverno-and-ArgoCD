## Why do we need Kyverno ?  (This is a work-in-progress Repo)
 
## Problem
#### Imagine a scenario where we have a Node with 5GB Memory and  accidentally  deployed a pod with exactly 5GB Memory. In that case no other pod would be deployed and all of them  would be in pending state

## Solution
#### So Kyverno is an Advanced Admission Controller where it evaluates, based on rules whether to deploy certain resoucres or not. In our case if we enforce a Kyverno policy that sets resources  limit to not cross  say 2GB, if we deployed a pod with 5GB Memory it would reject our request
We created two Kyverno Policies
1. Enforce pod-requests-and-limits --> This policy makes sure that each Deployment is created with resource  limits
   
![Screenshot (1492)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/3a9559d2-1691-43c7-b224-d35dd37454c5)

2. Enforce memory-limit --> This policy ensures that memory limts in a pod don't cross a certain value

    Example YAML file where memory specified is greater than the limit 

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod-exceed-limits
  name: pod-exceed-limits
spec:
  containers:
  - image: nginx
    name: pod-exceed-limits
    resources: 
      requests:
        memory: "4Gi" # Should not exceed 2GB
        cpu: "250m"
      limits:
        memory: "5Gi"
        cpu: "500m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

![Screenshot (1494)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/96ef7051-3b41-4a85-9714-5afa3a0c2208)


#### Using Kyverno, we can
1. Generate --> For example, Create a default network policy whenever a namespace is created.

2. Validate --> For example, Enforce CPU and Memory Resource Limits

3. Mutate --> For example, Add Sidecar Container for Logging to Deployments

4. Verify Images -->  For example, Verify if the Images used in the pod resources are properly signed and verified images.
## Architecture of Kyverno
![Untitled Diagram drawio(1)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/99dc40e1-42cd-452d-8835-df1773a7adb9)

### The required Kyverno policy is written and pushed to Git repo. ArgoCD deploys it onto the Kubernetes Cluster



## Install Kyverno Using Manifest Files
```
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.8.5/install.yaml
```

## ArgoCD
![Untitled Diagram drawio(2)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/359fb009-d48e-48e5-9906-6894eab1f2b7)

### Components of ArgoCD
#### Repo Server        --> This is used to watch the state of the version control system like github
#### Application Server --> This watches the state of Kubernetes Cluster
#### API Server --> It is useful for interacting with ArgoCD
#### Dex --> Provides authentication 
#### Redis --> Used for caching


## ArgoCD Insallation Using Helm

```
 helm repo add argo https://argoproj.github.io/argo-helm

 helm install argo argo/argo-cd

```
## Testing 

### We create a sample deployment with two replicas.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx1
  name: nginx1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx1
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

```
### Two pods are running below
![Screenshot (1495)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/8dcd2789-a47c-4bb9-9d93-6835642aaf5c)

### ArgoCD is watching the state of our repo and is in sync
![Screenshot (1498)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/46b82489-0717-447c-bedf-6e8d2cc3e937)


### In our git repo, i increased the replcias to 5
![Screenshot (1497)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/7df01408-8006-4b92-abc9-ff0effe67658)

### Result
![Screenshot (1499)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/9c990449-7897-4ced-b3f1-9657d5130c8e)



