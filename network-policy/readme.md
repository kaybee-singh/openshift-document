# Network Policy in Openshift

1. To isolate pods, network policy can be used. It can isolate pods from other pods running in the same namespace and other namespaces.
2. To configure network policy one does not need admin privileges. It gives developers more control over application.
3. It controls access between pods on the basis of labels if compared with traditional firewall which uses ip address for this.

First lets see how to label a namespace. Following command with further parameters can be used to set it.
```bash
oc label namespace
```
For instance with below command we are setting label on **test** namespace
```bash
oc label namespace test network=network-isolation
```
### Different network policies implementations.

1. Denying all the traffic for a namespace. Following policy will deny all the traffic to pods, including pods in the same namespace and other namespaces.
Create two projects
```bash
oc new-project pol1
oc new-project pol2
```
Create two deployments with httpd image in pol1
```bash
oc project pol1
oc create deployment httpd1-pol1 --image=registry.redhat.io/rhel8/httpd-24
oc create deployment httpd2-pol1 --image=registry.redhat.io/rhel8/httpd-24
```
Get the Pod IP details
```bash
oc get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE    IP             NODE                  NOMINATED NODE   READINESS GATES
httpd1-pol1-776f659d5f-j82pv   1/1     Running   0          2m5s   10.131.0.232   worker-0.example.com   <none>           <none>
httpd2-pol1-67c7bff97b-q7g4p   1/1     Running   0          73s    10.131.0.233   worker-0.example.com   <none>           <none>
```
Try to curl httpd1-pol1 from httpd2-pol1 with IP address of httpd1-pol1 pod fetched from above command. It will work
```bash
oc rsh httpd2-pol1-776f659d5f-j82pv
sh-4.4$ curl 10.131.0.232:8080
<head>
Output Omitted
</head>
```
Now create below network policy in a file denyall.yaml
```bash
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: denyall
spec:
  podSelector: {}
  ingress: []
```
Apply the policy
```bash
oc apply -f denyall.yaml
```
Now again try to access the httpd1-pol1 pod from httpd2-pol1 with curl. It will fail this time. So our network policy is working as expected.
```bash
oc rsh httpd2-pol1-776f659d5f-j82pv
sh-4.4$ curl 10.131.0.232:8080
^C
```
To only allow traffic from same namespace in which we are creating the policy.

```bash
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-from-same-namespace
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}
```
### 3. Now allow traffice from all the pods in pol1 namespace.

3.1 Create a new policy with name allow-pol1.yaml
```bash
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-pol1
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}
```
3.2 Now apply the yaml file.
```bash
oc apply -f allow-pol1.yaml
```
3.3 Now check the communication to httpd1-pol1 pod from httpd2-pol1.
```bash
oc rsh httpd2-pol1-776f659d5f-j82pv
sh-4.4$ curl 10.131.0.232:8080
<head>
Output Omitted
</head>
```
### 4. Create the policy to allow traffic from pol2 pods.
4.1 Create two deployments in pol2 namespace.
```bash
oc project pol2
oc create deployment httpd1-pol2 --image=registry.redhat.io/rhel8/httpd-24
oc create deployment httpd2-pol2 --image=registry.redhat.io/rhel8/httpd-24
```
4.2 Try to access the httpd-pol1 from httpd1-pol2 pod. 
=> For this first lets get the IP address of httpd-pol1
```bash
oc get pods -o wide -n pol1
NAME                           READY   STATUS    RESTARTS   AGE    IP             NODE                  NOMINATED NODE   READINESS GATES
httpd1-pol1-776f659d5f-j82pv   1/1     Running   0          2m5s   10.131.0.232   worker-0.example.com   <none>           <none>
httpd2-pol1-67c7bff97b-q7g4p   1/1     Running   0          73s    10.131.0.233   worker-0.example.com   <none>           <none>
```
=> Now try to curl the httpd-pol1 IP address from httpd1-pol2 pod. It would not work so close with Ctrl+C
```bash
oc get pods
NAME                           READY   STATUS    RESTARTS   AGE
httpd1-pol2-547ccfdb4b-fp5gs   1/1     Running   0          32s
httpd2-pol2-7d55756789-gvq7c   1/1     Running   0          13s
oc rsh httpd1-pol2-547ccfdb4b-fp5gs
sh-4.4$ curl 10.131.0.232:8080
^C
```
Set label on pol2, we will use it in the network-policy yaml.
```bash
oc label namespace pol2 project=pol2
```
List tha pod labels for both namespaces.
```bash
oc get pods -n pol1 --show-labels          
NAME                           READY   STATUS    RESTARTS   AGE   LABELS
httpd1-pol1-776f659d5f-j82pv   1/1     Running   0          61m   app=httpd1-pol1,pod-template-hash=776f659d5f
httpd2-pol1-67c7bff97b-q7g4p   1/1     Running   0          60m   app=httpd2-pol1,pod-template-hash=67c7bff97b
oc get pods -n pol2 --show-labels
NAME                           READY   STATUS    RESTARTS   AGE   LABELS
httpd1-pol2-547ccfdb4b-fp5gs   1/1     Running   0          33m   app=httpd1-pol2,pod-template-hash=547ccfdb4b
httpd2-pol2-7d55756789-gvq7c   1/1     Running   0          32m   app=httpd2-pol2,pod-template-hash=7d55756789
```
We can use app=httpd1-pol1 and app=httpd1-pol2 for pods respectively. 
Here httpd1-pol1 is sourcem we are creating the policy in httpd1-pol1's namespace. Destination is httpd2-pol2 we are creating the policy to allow for that pod from pol2 namespace.
4.3 Lets create the allow-httpd1-pol2.yaml policy file to allow Traffic to httpd-pol1 from httpd1-pol2 pod.
```bash
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-pod-and-namespace-both
spec:
  podSelector:
    matchLabels:
      app: httpd1-pol1
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            project: pol2
        podSelector:
          matchLabels:
            app: httpd1-pol2
      ports:
      - port: 8080
        protocol: TCP
```
Now apply the yaml and then check the pod access.
```bash
oc apply -f allow-httpd1-pol2.yaml
oc project pol2
```
Take shell of httpd1-pol2 pod and try to curl IP address of httpd1-pol1
```bash
oc get pods
NAME                           READY   STATUS    RESTARTS   AGE   
httpd1-pol2-547ccfdb4b-fp5gs   1/1     Running   0          33m   
httpd2-pol2-7d55756789-gvq7c   1/1     Running   0          32m
oc rsh httpd1-pol2-547ccfdb4b-fp5gs
sh-4.4$ curl 10.131.0.232:8080
<head>
Output Omitted
</head>
```
