# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file is where you record what you did.

What's here:

| File                 | What it is                                               |
| -------------------- | -------------------------------------------------------- |
| `src/`, `index.html` | a small React page that shows which Pod answered         |
| `nginx.conf`         | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile`         | builds the page, then serves it with nginx on port 80    |

You don't change any of these. You add `hello-app.yaml` and fill in the sections below.

## Before you push, check

- [ ] `hello-app.yaml` is in the root of this repository
- [ ] `screenshot.png` is in the root of this repository
- [ ] every section below is filled in

---

## 1. My image

Docker Hub image I deployed (for example `ravi123/hello-app:v1`):

```
capr2003/hello-app:v1
```

## 2. Pods across nodes

Output of `kubectl get pods -o wide` after you applied `hello-app.yaml`:

```
NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
hello-app-67ccdd9596-jtb7v   1/1     Running   0          27s   10.244.1.9   desktop-worker    <none>           <none>
hello-app-67ccdd9596-p9b5v   1/1     Running   0          27s   10.244.2.7   desktop-worker2   <none>           <none>
hello-app-67ccdd9596-pzrds   1/1     Running   0          27s   10.244.2.8   desktop-worker2   <none>           <none>
```

## 3. The Service and its endpoints

Output of `kubectl get service hello-app` and `kubectl get endpointslices -l kubernetes.io/service-name=hello-app`:

```
NAME        TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
hello-app   LoadBalancer   10.96.31.16   172.19.0.7    8080:32298/TCP   31s

NAME              ADDRESSTYPE   PORTS   ENDPOINTS                          AGE
hello-app-4gj54   IPv4          80      10.244.1.9,10.244.2.7,10.244.2.8   35s
```

## 4. Browser

Add `screenshot.png`: the page at `http://localhost:8080` with at least two Pods in the "Answers so far" table.

## 5. The four traps

For each trap, write what you saw and why it happened, in two or three lines of your own words.

**Trap 1 — the local image.**
Kubernetes nodes operate independently and cannot access custom images stored only on your local machine's Docker daemon. As a result, the cluster attempts to pull the image from the default registry (Docker Hub), resulting in an `ImagePullBackOff` error because the custom image doesn't exist there. To fix this, you must push your image to an accessible container registry or load it directly into your cluster's nodes.

**Trap 2 — the selector typo.**
A typo in a Service's selector breaks the connection between the Service and your Pods because Kubernetes cannot find any resources with that incorrect label. This results in an empty EndpointSlice, leaving the Service with nowhere to route incoming traffic until the typo is corrected.

**Trap 3 — the wrong targetPort.**
Setting an incorrect `targetPort` causes the Service to route traffic to a port on the Pod where no application is actively listening. Even though the EndpointSlice successfully maps to your Pods, the connection will fail because the containerized app is expecting traffic on its actual configured port (like port 80 for nginx).

**Trap 4 — the EXTERNAL-IP address.**
The EXTERNAL-IP provided by Docker Desktop belongs to its isolated internal network and cannot be routed directly from your host machine's browser. To access the service, you must use http://localhost:8080 because Docker Desktop automatically forwards LoadBalancer ports to your local machine. In a production cloud environment, this workaround isn't needed as the assigned EXTERNAL-IP will be a genuinely accessible public address.

## 6. Your YAML vs the generated YAML

Name two fields in the `--dry-run=client -o yaml` output that were not in your file. Why does your file still work without them?

- strategy: {}
- resources: {}
