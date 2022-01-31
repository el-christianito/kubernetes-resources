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
