# Kubernetes

## Overview

- Node = VM
- Pod = Container-Bucket or -Frame
- Node > Pod > Container

## Pods

### General

- a Pod cannot span several Nodes
- usually: one container per Pod
- possible: several tightly coupled containers in one Pod

### Networking

- Containers within pods share same network namespace (IP/Port)
- Containers within pods can talk to each over over localhost
- Container processes within pods need to bind to different network ports within pod, i.e.
  - Pods get unique IP address within a node
  - Containers within Pod need unique IP adresses

## Kubernetes Dashboard

1. copy first secret from here
   ```bash
   kubectl describe secret -n kube-system
   ```
2. start proxy
   ```bash
   kubectl proxy
   ```
3. paste secret here:
   http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

## CLI Commands

### Pod Commands

#### Manual Commands

- start a new pod

  ```bash
  kubectl run [pod-name] --image=[image-name]
  ```

- check all/pods/services

  ```bash
  kubectl get all[/pods/services]
  ```

- check specific pod, output YAML

  ```bash
  kubectl get pod [pod-name] -o yaml
  ```

- add access to a pod

  ```bash
  kubectl port-forward [pod-name] [local-port]:[container-port]
  ```

- delete a pod

  ```bash
  kubectl delete pod [pod-name]
  kubectl delete -f [path-to-yaml]
  ```

#### YAML file Commands

- create a pod using YAML file (--save-config for allowing apply call below)
- using apply call below does the same if pod does not exist

  ```bash
  kubectl create -f [path-to-yaml] --save-config
  ```

- create/update a pod using YAML file

  ```bash
  kubectl apply -f [path-to-yaml]
  ```

- manually edit deployed Pod

  ```bash
  kubectl edit pod [pod-name]
  kubectl edit -f [path-to-yaml]
  ```

- manually change single value of deployed Pod

  ```bash
  kubectl patch [pod-name]
  ```

#### Troubleshooting Commands

- get information on a pod
  ```bash
  kubectl describe pod [pod-name]
  ```
- execute shell in a pod
  ```bash
  kubectl exec [pod-name] -it -- sh
  ```
- shell into pod and URL (using cURL), add -c [containerID] in cases where multiple containers are running in Pod
  ```bash
  kubectl exec [pod-name] -- curl -s http://podIP
  ```

#### Deployment commands

- start deployment

  ```bash
  kubectl create -f [path-to-deployment-yaml] --save-config
  kubectl apply -f [path-to-deployment-yaml]
  ```

- list deployments with labels/by labels

  ```bash
  kubectl get deployment --show-labels
  kubectl get deployment -l app=nginx
  ```

- delete deployment and all associated Pods/Containers

  ```bash
  kubectl delete deployment [deployment-name]
  kubectl delete -f [path-to-deployment-yaml]
  ```

- scale Pods horizontally

  ```bash
  kubectl scale deployment [deployment-name] --replicas=5
  kubectl scale -f [path-to-deployment-yaml] --replicas=5
  ```

- expose port of container behind deployment

  ```bash
  kubectl port-forward deployment/[deployment-name] [local-port]:[container-port]
  ```

#### Service commands

- start service

  ```bash
  kubectl create -f [path-to-service-yaml] --save-config
  kubectl apply -f [path-to-service-yaml]
  ```

- list services with labels/by labels

  ```bash
  kubectl get service --show-labels
  kubectl get service -l app=nginx
  ```

- delete services and all associated Pods/Containers

  ```bash
  kubectl delete service [service-name]
  kubectl delete -f [path-to-service-yaml]
  ```

- expose port of service

  ```bash
  kubectl port-forward service/[service-name] [local-port]:[container-port]
  ```

## YAML Declarations

### Defining a Pod with YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
    - name: my-nginx
      image: nginx:alpine
```

## Pod Health

### Probes

- Liveness probes -> Is pod healthy and running as expected?
- Readiness probes -> Is a pod ready to receive requests?
- Possible results:
  - Success
  - Failure
  - Unknown

### Default behaviour

- if a pod container failed, it will be recreated by default (**restartPolicy** defaults to **Always**)

## Deployments

### Overview

- ReplicaSet is declarative way to manage Pods
- wraps ReplicaSets, "declarative way to manage Pods" using ReplicaSets
- ReplicaSets came before Deployments in Kubernetes History
- defines "Target" of Nr. Pods etc

### ReplicaSets

- Acts as a Pod controller for
  - self-healing
  - ensures requested number of Pods are available
  - fault-tolerance
  - scaling
  - relies on Pod template
  - no need to create Pods directly anymore!

### Deployments

- scales ReplicaSets, which scale Pods
- supports zero-downtime by creating and destroying ReplicaSets
- Rollback of deployments
- creates unique label being assigned to ReplicaSets and generated Pods
- very similar YAML to ReplicaSet

#### Deployment options

- Rolling Updates
- Blue-green deployments (multiple versions running in parallel)
- Canary deployments (small amount of traffic goes to new)
- Rollbacks

#### Defining a deployment with YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
spec:
  replicas: 2
  selector: # Pod template label(s)
  template: # template used to create the Pods
    spec:
      containers:
        - name: my-nginx
          image: nginx:alpine
```

- template can be extracted into separate file, being matched by labels

## Services

### Overview

- Services provides single point of entry for accessing one or more Pods
- abstract Pod IP addresses
- Load balances between pods
- relies on labels to associate a Service with a Pod
- Node's kube-proxy creates a virtual IP for services
- Services don't die & get recreated (in contrast to Pods)
- creates endpoints between Services and Pods

### Services Types:

- ClusterIP (Default)
  - Expose Service IP internally within the cluster
  - only Pods within cluster can talk to the service
  - allows Pods to call other Pods (without knowing IP address)
  - call possible via Service-IP or DNS (e.g. http://nginx-clusterip:[port])
  - load balances calls inside cluster
- NodePort
  - Expose service on each Node's IP at a static port
  - allocates a port from range (30000-32767)
  - external caller can talk to Node via this allocated port
  - very useful for debugging -> "proxying into a Pod"
- LoadBalancer
  - Provision external IP to act as load balancer
  - exposes service externally
  - useful when combined with cloud provider's load balancer
- ExternalName
  - maps Service to DNS name or IP address
  - proxies to an external service
  - acts as an alias for an external service
  - external service details are hidden from a cluster (easier to change)

### Defining a service with YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
  labels:
    app: nginx-clusterip
spec:
  type: ClusterIP # type of service
  selector: # pod template labels which the service will apply to
    tier: frontend
  ports: # define container target port and port for the service
    - port: 8080
      targetPort: 80
```
