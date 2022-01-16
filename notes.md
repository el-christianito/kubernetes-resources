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

### Troubleshooting Commands

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

### Deployment commands

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

### Service commands

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

### Volume commands

- get storages
  ```bash
  kubectl get pv --show-labels
  kubectl get pv -l app=nginx
  ```

### ConfigMap commands

- get all ConfigMaps

  ```bash
  kubectl get cm
  ```

- get specific ConfigMap

  ```bash
  kubectl get cm [configmap-name] -o yaml
  ```

- create ConfigMap from manifest
  ```bash
  kubectl create -f [path-to-manifest]
  ```
- create ConfigMap from config/data file (key/value pairs)
- filename becomes the key (see "ConfigMap manifest in YAML" below)

  ```bash
  kubectl create configmap [configmap-name] --from-file=[path-to-file]
  ```

- create ConfigMap from env file (key/value pairs)

  ```bash
  kubectl create configmap [configmap-name] --from-env-file=[path-to-file]
  ```

- create config map from passed data values

  ```bash
  kubectl create configmap [configmap-name]
    --from-literal=[key]=[value]
    --from-literal=[otherKey]=[otherValue]
  ```

- delete ConfigMap

  ```bash
  kubectl delete cm [configmap-name]
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

## Storage options

### Overview

How to store application state/data and exchange it between Pods?

- Volumes -> can hold data and state for Pods and containers
  - emptyDir -> empty directory for storing transient data (sharing a Pod's lifetime) between containers within a Pod
  - hostPath -> Pod mounts into the node's filesystem
- PersistentVolumes
  - nfs -> network file system share mounted into pod
  - configMap/secret -> special types of volumnes which provide a Pod with access to Kubernetes resources
  - cloud -> cluster-wide storage
- PersistentVolumeClaims
  - provides Pods with a more persistent storage option which is abstracted from details
- Storage classes
  - type of storage template which can be used to dynamically provision storage

### Volumes

How to find out mounted volumes?

```bash
kubectl describe pod [pod-name]
kubectl get pod [pod-name] -o yml
```

### Defining a volume within a YAML

#### Empty directory

- both defined containers are talking to same storage
- volume is lost when Pod goes down

```yaml
apiVersion: v1
kind: Pod
spec:
  volumes: # define initial volume named "html" being an empty directory
    - name: html
      emptyDir: {}
  containers:
    - name: nginx
      image: nginx:alpine
      volumeMounts:
        - name: html # must match name of defined volume
          mountPath: /usr/share/nginx/html
          readOnly: true
    - name: html-updater
      image: alpine
      command: ["/bin/sh", "-c"]
      args:
        - while true; do date >> /html/index.html;
          sleep 10; done
      volumeMounts:
        - name: html # must match name of defined volume
          mountPath: /html
```

#### HostPath volume

- easy to set-up
- good to get started when having single worker node
- when Node goes down, volume is lost -> for multi-node clusters network storage should be used

```yaml
apiVersion: v1
kind: Pod
spec:
  volumes:
    - name: docker-socket # define socket volume on host pointing to docker socket
      hostPath: # talk to specific directory on worker node
        path: /var/run/docker.sock
        type: Socket # also possible File/Directory/...
  containers:
    - name: docker
      image: docker
      command: ["sleep"]
      args: ["100000"]
      volumeMounts:
        - name: docker-socket
          mountPath: /var/run/docker.sock
```

#### (Azure) Cloud volume (similar on others)

```yaml
apiVersion: v1
kind: Pod
spec:
  volumes: # define volume named data
    - name: data
      azureFile:
        secretName: <azure-secret>
        shareName: <share-name>
        readOnly: false
  containers:
    - name: my-app
      image: someimage
      volumeMounts:
        - name: data # note: referencing name
          mountPath: /data/storage
```

### Persistent Volumes and PersistentVolumeClaims

#### PersistentVolume (PV)

- cluster-wide storage unit provisioned by an administrator with lifecycle **independent** from a Pod
- available to Pod even if it gets rescheduled on a different Node
- relies on a form of NAS (e.g. NFS, cloud, etc.)
- associated with a Pod by using PersistentVolumeClaim (PVC)

#### PersistentVolumeClaim (PVC)

- is a request for a storage unit (PV)
- Pod volume references the PVC
- Kubernetes relays it to PV

#### YAML

- there's a github with examples https://github.com/kubernetes/examples

PV:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity: 10Gi
  accessModes:
    - ReadWriteOnce # One client/Pod can mount for rw
    - ReadOnlyMany # many clients/Pods can mount for ro
  persistentVolumeReclaimPolicy: Retain # even if claim is deleted, don't delete
  azureFile:
    secretName: <azure-secret>
    shareName: <share-name>
    readonly: false
```

PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-dd-account-hdd-5g
  annotations:
    volume.beta.kubernetes.io/storage-class: accounthdd
spec:
  accessModes:
    - ReadWriteOnce # One client/Pod can mount for rw
  resources:
    requests:
      storage: 5Gi # only 5 Gig for this claim, eventhough PV is bigger
```

Volume using PVC:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-uses-account-hdd-5g
  labels:
    name: storage
spec:
  volumes:
    - name: blobdisk01
      persistentVolumeClaim: # bind to PVC
        claimName: pv-dd-account-hdd-5g
  containers:
    - image: nginx
      name: az-c-01
      command:
        - /bin/sh
        - -c
        - while true; do echo ($date) >> /mnt/blobdisk/outfile; sleep 1; done
      volumeMounts:
        - name: blobdisk01 # link volume name
          mountPath: /mnt/blobdisk
```

### Storage Classes

- can be used to defined different classes of storage
- acts as storage template
- supports dynamic provisioning of PV
- usually done by administrator (who don't have to create PVs in advance then)

#### YAML

StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
reclaimPolicy: Retain # retain or delete (default) after PVC is released, PVCs and PVs inerhit reclaim policy here
provisioner: kubernetes.io/no-provisioner # volume plugin that will be used to create PersistentVolume resource (here: none, so has PV to be done manually in advance)
volumeBindingMode: WaitForFirstConsumer # wait unit Pod making PVC is created. (default: Immediate, created once PVC is created)
```

PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity: 10Gi
  volumeMode: Block
  accessModes:
    - ReadWriteOnce # one client can mount for rw
  storage-class: local-storage # reference storage-class name from above
  local:
    path: /data/storage # path where data is stored on worker node
  nodeAffinity:
    required:
      nodeSelectorTerms: # select Node where the local storage PV is created
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - <node-name> # insert hostname here
```

PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce # access mode and storage classification PV needs to support
  storageClassName: local-storage # reference storage-class name from above
  resources:
    requests:
      storage: 1Gi
```

Use PVC

```yaml
apiVersion: v1
kind: [Pod | StatefulSet | Deployment]
---
spec:
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: my-pvc
```

## ConfigMaps

- a way to store configuration info and provide it to containers
- injects configuration data into a container
- can store entire files or provide key/value pairs
  - file: key is filename, value is contents (JSON, XML, key/values, ...)
  - provide on command-line
  - ConfigMap provided via manifest (YAML)
- ConfigMaps can be accessed from a Pod by
  - environment variables (key/value)
  - ConfigMap volume (as files)

### ConfigMaps in YAML

- manifest

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-settings # for referencing in Pods etc.
    labels:
      app: app-settings
  data:
    enemies: aliens
    lives: "3"
    enemies.cheat: "true"
    enemies.cheat.level: "noGoodRotten"
  ```

- config file (after importing config file via kubectl, this is what we have then)

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-settings # for referencing in Pods etc.
    labels:
      app: app-settings
  data:
    game.config: |- # file-name is key of the values
      enemies=aliens
      lives=3
      enemies.cheat=true
      enemies.cheat.level=noGoodRotten
  ```

- env file (after importing config file via kubectl, this is what we have then)
- notice this is much closer to manifest (file name is **not** included as key)

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-settings # for referencing in Pods etc.
    labels:
      app: app-settings
  data: enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
  ```

### Defining key/value pairs in a file

- config file instead of manifest, e.g. game.config

  ```
  enemies=aliens
  lives=3
  enemies.cheat=true
  enemies.cheat.level=noGoodRotten
  ```

### Defining key/value pairs in an Env file

- this is not a config file but an environment variables file (game-config.env)
- looks exactly the same (obviously)
- create call is different (see under Commands)

  ```
  enemies=aliens
  lives=3
  enemies.cheat=true
  enemies.cheat.level=noGoodRotten
  ```

### Using ConfigMaps

- access configMap in Pod, set one value to single environment variable

  ```yaml
  apiVersion: v1
  kind: Pod
  ---
  spec:
    template:
      containers: ...
      env:
        - name: ENEMIES # name of the environment variable created within pod
          valueFrom:
            configMapKeyRef:
              name: app-settings # here's our reference
              key: enemies
  ```

- access configMap in Pod, set whole ConfigMap as environment variables

  ```yaml
  apiVersion: v1
  kind: Pod
  ---
  spec:
    template:
      containers: ...
      envFrom:
        - configMapRef:
            name: app-settings # here's our reference
  ```

- access configMap in Pod, by volume
- each key is converted to a file, value is added into the file (so linuxy)
- nice:

  - if ConfigMap changes, referenced files would automatically update
  - if code detects file changes -> make ^s live updates possible! (30-60s delay possible though)

  ```yaml
  apiVersion: v1
  kind: Pod
  ---
  spec:
    volumes:
      - name: app-config-vol
        configMap:
          name: app-settings # here's our reference
    containers:
      volumeMounts:
        - name: app-config-vol # referencing volume above
          mountPath: /etc/config # path to store config files to be accessed
  ```

#### Defining key/value pairs in a file

- config file instead of manifest, e.g. game.config

  ```
  enemies=aliens
  lives=3
  enemies.cheat=true
  enemies.cheat.level=noGoodRotten
  ```

#### Defining key/value pairs in an Env file

- this is not a config file but an environment variables file (game-config.env)
- looks exactly the same (obviously)
- create call is different (see under Commands)

  ```
  enemies=aliens
  lives=3
  enemies.cheat=true
  enemies.cheat.level=noGoodRotten
  ```

#### Using ConfigMaps
