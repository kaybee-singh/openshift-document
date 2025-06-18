- Create the template yaml file.
```bash
vim template-1.yaml
```
```bash
apiVersion: template.openshift.io/v1
kind: Template
metadata:
 name: pod-template1
 namespace: temp
objects:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app: test
    name: test
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: test
    template:
      metadata:
        labels:
          app: test
      spec:
        containers:
        - image: --image=registry.redhat.io/rhel8/httpd-24
          name: httpd-24
```
- Then create the template
```bash
oc create -f template-1.yaml
```
- Then create the resources from the template
```bash
oc process pod-template1|oc create -f
````
- Now lets try to list if resources are created.
```bash
oc get all -n temp 
```
