# Kubernetes advanced

## Multi-container Pods

### Introduction

Pods add functions to containers like:

- probes
- affinities
- restart policies
- termination control

### Init Pattern

- container which starts before app container
- runs once
- initialize environment for app container, e.g. git pull, setting up stuff, ...
- main app container can be kept clean, initialization logic decoupled
- allows single responsibility principle
- recovering/init databases is probably better solved by Kubernetes operators, but for simple tasks this is awesome
- several init containers declarable:
  - all running sequentially
  - all have to finish successfully before starting any app container
  - Pod restarts if init container fails
  - make sure to run idempotent code in init containers!
- set resource requests & limits
  - keem them low for init containers
  - only the biggest init container is counted!

#### YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ps-init
  labels:
    app: initializer
spec:
  initContainers: # has to finish successfully before container below starts
    - name: init-ctr
      image: busybox
      command:
        [
          "sh",
          "-c",
          "until nslookup pluralsight-ftw; do echo waiting for pluralsight-ftw service; sleep 1; done; echo Service found!",
        ]
  containers:
    - name: web-ctr
      image: nigelpoultan/web-app:1.0
      ports:
        - containerPort: 8080
```

### Sidecar Pattern

- long running
- helper for the main app container
- single-purpose
- one use-case in relation to init-example above: what if we regularly want to sync changes from git?
- sidecars can be a pain to schedule (no guarantee which container runs first)
  - currently no way to set
  - is planned in new k8s versions
- declaration in YAML just as another entry in the containers-array

### Adapter Pattern

- specialized version of sidecar pattern
- e.g. format mismatch for external service -> conversion nessary to standardize output -> adapter
- accessible in same host, different IP to collect from e.g. monitoring service (e.g. Prometheus)

### Ambassador Pattern

- sits between application container and external system
- while main app focusses on single task, ambassador brokers external connections
