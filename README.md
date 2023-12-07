## Why do we need Kyverno ?  

 
## Problem
#### Imagine a scenario where we have a Node with 5GB Memory and  accidentally  deployed a pod with exactly 5GB Memory. In that case no other pod would be deployed and all of them  would be in pending state

## Solution
#### So Kyverno is an Advanced Admission Controller where it evaluates, based on rules whether to deploy certain resoucres or not. In our case if we enforce a Kyverno policy that sets resources  limit to not cross  say 2GB, if we deployed a pod with 5GB Memory it would reject our request



## High Level Design
![Untitled Diagram drawio(1)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/99dc40e1-42cd-452d-8835-df1773a7adb9)

To explain briefly, we write our kyverno policy and push it github repo. Since ArgoCD is configured to watch for changes in the Repo and sync it with the cluster, ArgoCD will deploy the necessary changes to our Namespace.

## The Heart of Kyverno
Kyverno is a policy engine designed for Kubernetes

A Kyverno policy is a collection of rules. Each rule consists of a match declaration, an optional exclude declaration, and one of a validate, mutate, generate, or verifyImages declaration. Each rule can contain only a single validate, mutate, generate, or verifyImages child declaration.



![Kyverno-Policy-Structure](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/a8b4632d-3748-416b-8338-d49e79434aab)

Policies can be defined as cluster-wide resources (using the kind ClusterPolicy) or namespaced resources (using the kind Policy.) As expected, namespaced policies will only apply to resources within the namespace in which they are defined while cluster-wide policies are applied to matching resources across all namespaces. Otherwise, there is no difference between the two types.


## Install Kyverno Using Manifest Files
```
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.8.5/install.yaml
```


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


### The required Kyverno policy is written and pushed to Git repo. ArgoCD deploys it onto the Kubernetes Cluster





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

### Case1. Auto-Sync 

Our Cluster has no pods deployed in the default namespace

![Screenshot (1504)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/9dd0b7ea-f331-44ee-92eb-a44743c1fec2)

We upload a deployment yaml file in our git repo 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  selector:
    matchLabels:
      app: nginx-app
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: ng
        image: nginx
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"

# Seperate files in github
---
apiVersion: v1
kind: Service
metadata:
  name: nginxpp-service
spec:
  selector:
    app: nginx-app 
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
We now proceed to build ArgoCD

1. Source - Our github link and the path is our folder name where yaml files are stored

2. Destination - Where our K8S cluster is running

3. Note - Selfheal and pruning are disabled by default

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample-test
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/satya19977/argo-kyverno-testing
    targetRevision: HEAD
    path: argo-kyverno
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    automated:
     selfHeal: true
     prune: true  
```
ArgoCD uses git as the single source of truth and synced our cluster with Git repo
![Screenshot (1507)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/85245cb4-ec3d-439e-834b-608dbbfaad04)

As Evident, we have all the resources defined in our Git
![Screenshot (1506)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/a8094de7-853d-4b22-acf5-9d9ed44c4dd6)

### Case2. Self-Healing

If this is enabled, any change made directly in the cluster isn't enforced.

Let's increase the number of replicas directly in the cluster directly and observer what happens

![Screenshot (1512)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/d22bcac5-7709-430b-9517-58b346d700ec)

![Screenshot (1513)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/6ea53525-1574-4b57-a19a-e5969330f10b)

Even though we manually made change to our cluster since selfheal is enabled, argocd will not push those changes

### Case3. Enable Prune

Since prune is enabled any change in the Git will also reflect in our cluster.
![Screenshot (1518)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/8cbe81d3-3290-48ba-923b-5fa598baafcf)

Changing the name of the deployment to deployment-app in the repo has been propgated to the cluster as well


![Screenshot (1517)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/47a96227-5d56-40fc-9c7c-03f713ae0169)

### Cae4. Let's deploy a pod with no limits and resources and see how kyverno reacts.

This is the yaml file in our git
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx
spec:
  selector:
    matchLabels:
      app: nginx-app
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: ng
        image: nginx
        ports:
        - containerPort: 8080
```
We see here that the pod has been deployed because in our kyverno policy we just set validatefailureaction to audit rathen than enforce. So that's why our pod has been deployed but the logs give us a warning to enforce resource limits in our pod

![Screenshot (1521)](https://github.com/satya19977/K8S-Security-With-Kyverno-and-ArgoCD/assets/108000447/585a5be8-7e9c-41b1-ac9a-ceeaa1a7d061)



