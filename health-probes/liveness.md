1. Create a pod which creates a directory and then deletes it after waiting for sometime.
```yaml
oc create -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: liveness-test1
spec:
  containers:
  - name: liveness
    image: busybox
    # Create a file, then sleep. After 30s, delete the file to trigger failure.
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
EOF
```
2. See if there are any pod restarts and liveness failures.
```bash
oc get pods -w
oc get events --sort-by='.lastTimestamp'
```

3. Now create a new pod with liveness probe and observe the difference.
```yaml
oc create -f - << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: liveness-test2
spec:
  containers:
  - name: liveness
    image: busybox
    # Create a file, then sleep. After 30s, delete the file to trigger failure.
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
EOF
```
4. Now observe the pod which has liveness probe configured and also check if there is any liveness failure.
```bash
oc get pods -w
oc get events --sort-by='.lastTimestamp'
```
5. Deployment yaml
```yaml
oc create -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: liveness-deployment1
  labels:
    app: liveness-test1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: liveness-test1
  template:
    metadata:
      labels:
        app: liveness-test1
    spec:
      containers:
      - name: liveness
        image: busybox
        # Logic: Create file, wait 30s, delete file.
        args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
        
        # This tells Kubernetes how to check if the app is "alive"
EOF
```
6. Now lets test the liveness probe with HTTP GET method.
```yaml
oc create -f - << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: http-probe-test
spec:
  containers:
  - name: nginx-container
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
    command: ["/bin/sh", "-c"]
    args:
      - |
        echo "OK" > /usr/share/nginx/html/health.html;
        nginx -g "daemon off;"
    livenessProbe:
      httpGet:
        path: /health.html
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
EOF
```
7. Now remote the health.html file and see if pod gets restarted due to liveness failure.
```bash
 oc exec http-probe-test -- rm /usr/share/nginx/html/health.html
oc get pods -w
oc get events --sort-by='.lastTimestamp'
```
8. Now to test the TCP Socket liveness probe create a new redis pod. That redis pod is running on Port no. 6379.
```yaml
oc create -f - << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: redis-liveness-demo
  labels:
    app: cache
spec:
  containers:
  - name: redis
    image: redis:alpine
    ports:
    - containerPort: 6379
    livenessProbe:
      tcpSocket:
        port: 6379
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 3
EOF
```
9. Lets shutdown the redis process in the pod.
```bash
oc exec redis-liveness-demo -- redis-cli shutdown
oc get pods -w
oc get events --sort-by='.lastTimestamp'
```
