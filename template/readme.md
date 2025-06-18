## Task 1 - First lets try to create a simple Template file without any parameters. We will create the resources from the template. This template file will create a simple deployment.

- Create the template yaml file.
```bash
vim template-1.yaml
```
- Below is file  
```yaml
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
```
- Now lets try to list if resources are created.
```bash
oc get all -n temp 
```
---

## Task 2 - Try creating the parameters in the template.  We can create 2 kinds of parameters

Optional
Required - Set the Required field in the parameter to True

Openshift can also generate random values for the parameters if used `generate: random` setting.

- In below yaml file we have a parameter specified. Parameter name is DEPLOY_NAME. If we do not specify the parameter while running the template then there is a value specified as defaultname.
```yaml
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
    name: ${DEPLOY_NAME}
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
parameters:
  - name: DEPLOY_NAME
    description: DEPLOY_NAME
    value: defaultname
```
- First delete the template
```bash
oc delete template pod-template1
```
- Now create the template
```bash
oc create -f template-1.yaml
```
- Now process the template without any parameter.
```bash
oc process pod-template1 |oc create -f -
```
- Now verify that pod and deployment is created with defaultname which is specified in the value.
```bash
oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                               READY   STATUS             RESTARTS   AGE
pod/defaultname-768669d98c-skbkt   0/1     InvalidImageName   0          4s
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/defaultname   0/1     1            0           4s
```
- Now run again and create the deployment with name defaultname1
```bash
oc process pod-template1  -p DEPLOY_NAME=defaultname1 |oc create -f -
```
- Now list and verify that pod and deployment is created with name `defaultname1`
```bash
oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                                READY   STATUS             RESTARTS   AGE
pod/defaultname-768669d98c-skbkt    0/1     InvalidImageName   0          5m52s
pod/defaultname1-768669d98c-pg2sj   0/1     InvalidImageName   0          12s
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/defaultname    0/1     1            0           5m52s
deployment.apps/defaultname1   0/1     1            0           12s
```
