- Create a deployment
```bash
oc create -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-startup
spec:
  replicas: 1
  selector:
    matchLabels:
      app: probe-startup
  template:
    metadata:
      labels:
        app: probe-startup
    spec:
      containers:
      - name: slow-app
        image: busybox
        # This script waits 60 seconds before creating the 'started' file
        command: ["sh", "-c", "sleep 60; touch /tmp/started; sleep 3600"]
        startupProbe:
          exec:
            command:
            - cat
            - /tmp/started
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          exec:
            command:
            - cat
            - /tmp/started
          periodSeconds: 5
EOF
```
- The pod will show 0/1 READY. Even if the livenessProbe is configured, it will not kill the pod during the first 60 seconds.
- Because the Startup Probe is "protecting" the container. It allows up to 300 seconds ($30 \text{ attempts} \times 10 \text{ seconds}$) for the app to start.
- Checking the Events during Startup,  You will see "Startup probe failed" warnings in the events.
- Unlike Liveness probes, a failing Startup probe doesn't trigger a restart until the failureThreshold is reached.
```bash
oc describe pod probe-startup-xxx
```
- The Startup Probe passes. Now, the Liveness Probe takes over. If you were to delete that file now, the Liveness probe would notice and restart the pod.
- 
