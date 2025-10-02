To configure an additional ingress controller first create a YAML file.
```bash
vim custom-ingress.yaml
```
Add below content.
```bash
apiVersion: operator.openshift.io/v1
kind: IngressController
metadata:
  name: custom-ingress
  namespace: openshift-ingress-operator
spec:
  domain: custom.lab.example.com
  replicas: 2
  endpointPublishingStrategy:
    type: HostNetwork
```
Apply the configuration.
```bash
oc apply -f custom-ingress.yaml
```
Edit the DNS forward file so that it can resolv the custom wildcard domain custom.lab.example.com. Add an additional entry for Ha proxy system with *.custom.lab.example.com. 192.168.122.22 is IP address of Bastion system on which my Ha proxy is running.
```bash
*.apps.lab.example.com.         IN      A       192.168.122.22
*.custom.lab.example.com.       IN      A       192.168.122.22
```
Restart the named
```bash
systemctl restart named
```
Now test it by creating a project, pod and route.
```bash
oc new-project custom-test
oc new-app --name=httpd-example registry.redhat.io/rhel8/httpd-24
```
Now expose the service to create a route with our custom URL. In below example out custom URL has a different domain than the default ingress controller domain.
```bash
oc create route edge httpd-example \
  --service=httpd-example \
  --hostname=httpd.custom.lab.example.com
```
Now list the created route
```bash
 oc get route
NAME            HOST/PORT                                 PATH   SERVICES        PORT       TERMINATION   WILDCARD
httpd-example   httpd.custom.lab.example.com ... 1 more          httpd-example   8080-tcp   edge          None
```
