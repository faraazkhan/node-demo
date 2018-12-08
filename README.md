# node-demo: Using Draft + Brigade for creating a Node.js app


This is a demonstration project that combines several develope tools to create a
multi-layered create/test/debug/deploy environment.

[![Recording of this Demo](https://img.youtube.com/vi/suNjNkhmWTs/0.jpg)](https://youtu.be/suNjNkhmWTs?t=20s)

## About the Demo

This demo uses the following:

- [Draft](https://draft.sh) for creating and running this application.
- [Brigade](https://brigade.sh) for executing various sequences of tests.
- any Kubernetes cluster

The idea is that when one runs `draft up`, Brigade silently runs in-cluster tests
against the created artifacts while Draft sets everything up for you to do manual
testing and debugging. This makes testing a transparent part of "inner loop"
development.

![inner loop](docs/images/inner-loop.png)

You can also configure this application so that upon a `git push` to GitHub, the "outer loop" testing is triggered, and
Brigade plays the role of a more traditional CI system.

![outer loop](docs/images/outer-loop.png)

## Setting Up the Demo

- deploy Brigade in your cluster:


```
$ helm install -n brigade ./charts/brigade --set rbac.enabled=true --set gw.enabled=false --set vacuum.enabled=false 
```

- create a new Brigade project - make sure to use the `<repo>/<image>` you are building here as the VCS Sidecar of your project:

```
$ brig project create
? Project name radu-matei/node-demo
? Full repository name github.com/radu-matei/node-demo
? Clone URL (https://github.com/your/repo.git) [? for help] (https://github.com/radu-matei/d? Clone URL (https://github.com/your/repo.git) https://github.com/radu-matei/node-demo.git
? Add secrets? No
Auto-generated a Shared Secret: "Cmwbe42gh1P3gD9ShFB3ANM9"
? Configure GitHub Access? No
? Configure advanced options Yes
? Custom VCS sidecar node-demo:edge
? Build storage size 
? Build storage class 
? Job cache storage class 
? Worker image registry or DockerHub org 
? Worker image name 
? Custom worker image tag 
? Worker image pull policy IfNotPresent
? Worker command yarn -s start
? Initialize Git submodules No
? Allow host mounts No
? Allow privileged jobs Yes
? Image pull secrets 
? Default script ConfigMap name 
? Upload a default brigade.js script 
Project ID: brigade-646bf4e98239dcddf6f67ba3df7a407bba9018c66e71a369ad62ef
```

> If you use a remote Kubernetes cluster make sure to push the image to your registry and change the appropriate registry/tag when creating the project, as well as to force Kubernetes to pull the new image in brigade.js

> If you are running on Minikube, you can pass the `--skip-image-push` flag to `draft up` (but make sure to `eval $(minikube docker-env)` before)

At this point, if you configured Brigade correctly, you can `draft up` (or `./draft.sh`, depending on what Draft version you are running):

```
$ draft up --skip-image-push
Draft Up Started: 'node-demo': 01CY7ZX1MQTH7BRFKKHQ6297BW
node-demo: Building Docker Image: SUCCESS ⚓   (1.0042s)
node-demo: Releasing Application: SUCCESS ⚓   (0.8531s)
Inspect the logs with `draft logs 01CY7ZX1MQTH7BRFKKHQ6297BW`
Event created. Waiting for worker pod named "brigade-worker-01cy7zx3q4gvtemwj9f89hkz59".
Build: 01cy7zx3q4gvtemwj9f89hkz59, Worker: brigade-worker-01cy7zx3q4gvtemwj9f89hkz59
prestart: no dependencies file found
prestart: loading script from /etc/brigade/script
[brigade] brigade-worker version: 0.18.0
{ buildID: '01cy7zx3q4gvtemwj9f89hkz59',
  workerID: 'brigade-worker-01cy7zx3q4gvtemwj9f89hkz59',
  type: 'image_push',
  provider: 'brigade-cli',
  revision: { commit: '', ref: 'master' },
  logLevel: undefined,
  payload: '{\n    "image": "node-demo:edge"\n}' }
[brigade:k8s] Creating secret test-01cy7zx3q4gvtemwj9f89hkz59
[brigade:k8s] Creating secret ftest-01cy7zx3q4gvtemwj9f89hkz59
[brigade:k8s] Creating pod test-01cy7zx3q4gvtemwj9f89hkz59
[brigade:k8s] Creating pod ftest-01cy7zx3q4gvtemwj9f89hkz59
[brigade:k8s] Timeout set at 900000
[brigade:k8s] Timeout set at 900000
[brigade:k8s] Pod not yet scheduled
[brigade:k8s] Pod not yet scheduled
[brigade:k8s] default/test-01cy7zx3q4gvtemwj9f89hkz59 phase Pending
[brigade:k8s] default/ftest-01cy7zx3q4gvtemwj9f89hkz59 phase Pending
[brigade:k8s] default/test-01cy7zx3q4gvtemwj9f89hkz59 phase Running
[brigade:k8s] default/ftest-01cy7zx3q4gvtemwj9f89hkz59 phase Running
[brigade:k8s] default/test-01cy7zx3q4gvtemwj9f89hkz59 phase Succeeded
[brigade:k8s] default/ftest-01cy7zx3q4gvtemwj9f89hkz59 phase Running
[brigade:k8s] default/ftest-01cy7zx3q4gvtemwj9f89hkz59 phase Running
done
```

## Questions

**How does the Draft image work as a sidecar and as an app?**

The entrypoint `start.sh` detects whether it is executing as a sidecar. If not,
it starts the node server.
