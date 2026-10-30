# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file records what was done.

What's here:

| File | What it is |
|---|---|
| `src/`, `index.html` | a small React page that shows which Pod answered |
| `nginx.conf` | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile` | builds the page, then serves it with nginx on port 80 |

The application files were not changed. `hello-app.yaml` was added for the Kubernetes Deployment and Service.

## Before you push, check

- [x] `hello-app.yaml` is in the root of this repository
- [x] `screenshot.png` is in the root of this repository
- [x] every section below is filled in

---

## 1. My image

Docker Hub image deployed:

```text
aaniruddhaa/hello-app:v1
```

## 2. Pods across nodes

After applying `hello-app.yaml`, the Deployment was configured with 4 replicas.

```text
NAME                         READY   STATUS    RESTARTS   AGE
hello-app-5869498957-6mh98   1/1     Running   0          ...
hello-app-5869498957-g9pjx   1/1     Running   0          ...
hello-app-5869498957-vxqzg   1/1     Running   0          ...
hello-app-5869498957-6hlp9   1/1     Running   0          ...
```

The four Pods were running successfully on the Kubernetes cluster.

## 3. The Service and its endpoints

The Service was created as a `LoadBalancer`:

```text
NAME        TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
hello-app   LoadBalancer   10.96.28.102   172.18.0.7    8080:30168/TCP
```

The EndpointSlice contained the Pod IP addresses and used port 80:

```text
NAME              ADDRESSTYPE   PORTS   ENDPOINTS
hello-app-l5z6z   IPv4          80      10.244.1.5,10.244.2.5,10.244.1.4
```

After changing the Deployment to 4 replicas, the Service continued to route requests to the running Pods.

## 4. Browser

The application was opened at:

```text
http://localhost:8080
```

The page displayed **Hello from Kubernetes** and showed four different Pods in the "Answers so far" table.

The screenshot is saved as:

```text
screenshot.png
```

## 5. The four traps

**Trap 1 — the local image.**

The locally built `hello-app:v1` image was used to create the first Deployment. In this Docker Desktop Kubernetes setup, the Pods were able to run the local image successfully instead of showing `ErrImagePull` or `ImagePullBackOff`. The image was then tagged and pushed to Docker Hub as `aaniruddhaa/hello-app:v1`, which was used for the final Deployment.

**Trap 2 — the selector typo.**

The Service selector was changed from `app: hello-app` to `app: helo-app`. The EndpointSlice then had no matching Pod endpoints because the Pod label was still `app: hello-app`. Changing the selector back to `app: hello-app` fixed the connection.

**Trap 3 — the wrong targetPort.**

The Service `targetPort` was temporarily changed from 80 to 8080. The EndpointSlice then showed port 8080, but the nginx application inside the Pods listens on port 80. Changing `targetPort` back to 80 restored the correct configuration.

**Trap 4 — the EXTERNAL-IP address.**

The LoadBalancer Service showed an EXTERNAL-IP address of `172.18.0.7`. For this Docker Desktop setup, the application was accessed using `http://localhost:8080` as specified in the lab instead of using the EXTERNAL-IP address.

## 6. Your YAML vs the generated YAML

Two fields present in the `--dry-run=client -o yaml` output but not in the manually written file were `creationTimestamp` and `status`.

The manually written YAML still works because Kubernetes supplies and manages default or runtime fields itself. The `status` field is maintained by Kubernetes after the resource is created, so it does not need to be written in the submitted manifest.
