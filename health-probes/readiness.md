Create a deployment with readiness probe configured on HTTP Path.
```yaml
cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-http
spec:
  replicas: 1
  selector:
    matchLabels:
      app: probe-http
  template:
    metadata:
      labels:
        app: probe-http
    spec:
      containers:
      - name: nginx
        image: bitnami/nginx:latest
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 5
EOF
```
- Run oc get pods. You will see 0/1 READY because /healthz doesn't exist in a default Nginx image.
```bash
oc get pods
NAME                         READY   STATUS    RESTARTS   AGE
probe-http-95ddf75fd-j7pzc   0/1     Running   0          3m3s
```
- Run oc describe pod probe-http-xxx. You will see Readiness probe failed: HTTP probe failed with statuscode: 404.
```bash
oc describe pod probe-http-95ddf75fd-j7pzc
10.573s including waiting). Image size: 108346255 bytes.
  Normal   Created         4m17s               kubelet            Created container: nginx
  Normal   Started         4m17s               kubelet            Started container nginx
  Warning  Unhealthy       2m3s (x25 over 4m)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
```
- Fix readiness probe Create the file inside the pod.
```bash
oc exec probe-http-95ddf75fd-j7pzc -- touch /opt/bitnami/nginx/html/healthz
```

Step D: Watch oc get pods. It will turn 1/1 READY within 5 seconds.
- Now create a TCP socket scenario.
```yaml
cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-tcp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: probe-tcp
  template:
    metadata:
      labels:
        app: probe-tcp
    spec:
      containers:
      - name: redis
        image: bitnami/redis:latest
        env:
        - name: ALLOW_EMPTY_PASSWORD
          value: "yes"
        readinessProbe:
          tcpSocket:
            port: 6379
          initialDelaySeconds: 5
          periodSeconds: 10
EOF
```
- Run oc get pods. Since Redis starts almost instantly, it should become 1/1 READY very quickly.
```bash
 oc get pods
NAME                         READY   STATUS    RESTARTS   AGE
probe-tcp-6d5d58b8d6-xrp5w   1/1     Running   0          23s
```
- To break it change the probe port to 6380 (a port Redis isn't using) via the OpenShift Console. Then Observe that the pod stays Running but READY becomes 0/1.
- 
- One more scenario which is running a command inside container.
```yaml
cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-exec
spec:
  replicas: 1
  selector:
    matchLabels:
      app: probe-exec
  template:
    metadata:
      labels:
        app: probe-exec
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sh", "-c", "sleep 30; touch /tmp/ready; sleep 3600"]
        readinessProbe:
          exec:
            command:
            - cat
            - /tmp/ready
          initialDelaySeconds: 5
          periodSeconds: 5
EOF
```
- Container starts, but the ready file is only created after 30 seconds.
- In oc get pods -w. we will see the pod in 0/1 for exactly 30 seconds before switching to 1/1.
- Run below to Watch the pod drop out of "Ready" status immediately.
```bash
oc exec probe-exec-pod -- rm /tmp/ready.
```
