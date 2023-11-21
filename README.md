## Why do we need Kyverno ?

## Problem
#### Imagine a scenario where we have a Node with 5GB Memory and  accidentally  deployed a pod with exactly 5GB Memory. In that case no other pod would be deployed and all of them  would be in pending state

## Solution
#### So Kyverno is an Advanced Admission Controller where it evaluates, based on rules whether to deploy certain resoucres or not. In our case if we enforce a Kyverno policy that sets resources  limit to not cross  say 2GB, if we deployed a pod with 5GB Memory it would reject our request
![Screenshot (1492)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/3a9559d2-1691-43c7-b224-d35dd37454c5)

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

## ArgoCD Insallation Using Helm

```
 helm repo add argo https://argoproj.github.io/argo-helm

 helm install argo argo/argo-cd

```


