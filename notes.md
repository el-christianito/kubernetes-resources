# Kubernetes

## Overview
* Node = VM
* Pod = Container-Bucket or -Frame
* Node > Pod > Container

## Pods
### General
* a Pod cannot span several Nodes
* usually: one container per Pod
* possible: several tightly coupled containers in one Pod

### Networking
* Containers within pods share same network namespace (IP/Port)
* Containers within pods can talk to each over over localhost
* Container processes within pods need to bind to different network ports within pod, i.e.
  * Pods get unique IP address within a node
  * Containers within Pod need unique IP adresses

## Kubernetes Dashboard
1. copy first secret from here
    ```bash
    kubectl describe secret -n kube-system
    ```
2. start proxy
    ```bash
    kubectl proxy
    ```
2. paste secret here:
  http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/


## CLI Commands
### Pod Commands
#### Manual Commands
* start a new pod
  ```bash
  kubectl run [pod-name] --image=[image-name]
  ```

* check all/pods/services
  ```bash
  kubectl get all[/pods/services]
  ```

* add access to a pod
  ```bash
  kubectl port-forward [pod-name] [local-port]:[container-port]
  ```

* delete a pod
  ```bash
  kubectl delete pod [pod-name]
  kubectl delete -f [path-to-yaml]
  ```

#### YAML/deployment Commands
* create a pod using YAML file (--save-config for allowing apply call below)
* using apply call below does the same if pod does not exist
  ```bash
  kubectl create -f [path-to-yaml] --save-config
  ```

* create/update a pod using YAML file
  ```bash
  kubectl apply -f [path-to-yaml]
  ```

* manually edit deployment
  ```bash
  kubectl edit pod [pod-name]
  kubectl edit -f [path-to-yaml]
  ```

* manually change single value of deployment
  ```bash
  kubectl patch [pod-name]
  ```

#### Troubleshooting Commands
* get information on a pod
  ```bash
  kubectl describe pod [pod-name]
  ```
* execute shell in a pod
  ```bash
  kubectl exec [pod-name] -it -- sh
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
* Liveness probes -> Is pod healthy and running as expected?
* Readiness probes -> Is a pod ready to receive requests?
* Possible results:
  * Success
  * Failure
  * Unknown

### Default behaviour
* if a pod container failed, it will be recreated by default (**restartPolicy** defaults to **Always**)