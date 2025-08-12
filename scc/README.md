# SCC

## What is SCC?

An SCC (Security Context Constraint) is a powerful OpenShift resource that controls the security parameters a pod can run with. 
It defines a set of permissions that a pod, and by extension its containers, must operate within. 
This includes things like what user ID or group a pod can run as, which volumes it can mount, and whether it can use a privileged container.

## Why SCCs Are Required in OpenShift?

OpenShift is designed with a strong "secure by default" philosophy, and SCCs are central to that. 
They enforce a multi-layered security model that goes beyond standard Kubernetes. 
For example, by default, OpenShift prevents containers from running as the root user and instead assigns them a random, non-privileged user ID from a pre-defined range. 
SCCs prevent unauthorized actions (e.g., running as root, accessing host resources) and protect against vulnerabilities, critical for your storage-related tasks (e.g., Image Registry, SSH pod debugging).

## Why It's NOT a Kubernetes Feature?

This is a common point of confusion. SCCs are an OpenShift-specific feature, not a native part of standard Kubernetes. 
In standard Kubernetes, the closest equivalent is Pod Security Standards (PSS), which was introduced to bring a similar level of security policy enforcement to the core project.

## What Are Its Benefits?

- `Proactive Security:` SCCs prevent security misconfigurations by blocking non-compliant pods from being scheduled in the first place.
- `Granular Control:` Administrators can create and assign different SCCs to different users or service accounts,
   allowing for fine-grained control over which teams can run privileged or more permissive workloads.
- `Enforcement of Best Practices:` By default, SCCs promote security best practices like preventing containers from running as root and limiting host access.

To list all scc in cluster run following command.
```bash
oc get scc
```
To understand how kubernetes and openshift runs the pods lets do a simple test.
Run the same nginx image on both Openshift and Kubernetes. We are using minikube here for k8s.
Running on OCP throws a warning.
```bash
oc run nginx --image=docker.io/nginx
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false),
unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```
If we look under the pod then it is running with O UID.
```bash
oc get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2m20s
oc rsh nginx
# id
uid=0(root) gid=0(root) groups=0(root)
```
Pod started in K8 directly without any warning and it is running with root
```bash
kubectl run hello --image=docker.io/nginx
kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
hello   1/1     Running   0          9m2s
 kubectl exec -it hello -- /bin/bash                                              
root@hello:/# id
uid=0(root) gid=0(root) groups=0(root)
```
In OCP in a particular namespace to check the pod security modes set on the cluster.
```bash
oc get namespace new -o yaml | grep pod-security.kubernetes.io
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
```
To fetch scc used by the pod run below.
```bash
 oc get pod nginx -o jsonpath='{.metadata.annotations.openshift\.io/scc}
anyuid
```
If we login with a non-admin user and try to run the same nginx pod then it will fail with permission error.
```bash
oc login -u bob -p redhat
oc new-project bob
oc run bobnginx --image=docker.io/nginx
oc get pods bobnginx
NAME       READY   STATUS             RESTARTS      AGE
bobnginx   0/1     CrashLoopBackOff   6 (72s ago)   7m49s
```
If we look at pod logs then permission denied error is visible
```bash
oc logs bobnginx
2025/08/12 06:11:32 [warn] 1#1: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
2025/08/12 06:11:32 [emerg] 1#1: mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
```
This pod id created with restricted-v2 which can be checked with below command.
```bash
oc get pod bobnginx -o jsonpath='{.metadata.annotations.openshift\.io/scc}'
restricted-v2
```
FAQs

Which SCC is used for pods?
Most pods will have the `restricted-v2` SCC by default. This SCC provides the limited access to resources. 

Why container images pushed from docker.io and other third party registeries fail to run on Openshift?

Those images when executed with a non-admin user may fail as those will be started with `restricted-v2` SCC. If a container image requires running the process with a particular user ID then it may fail because restricted-v2 runs the container by using a random user id.

How to set SCC for a pod?
It is not possible to set SCC for a Pod. To do it we need to create a new service account. Then associate that particular SCC with that service account.
In below example we have added the anyuid to the testsa service account.
NOTE:- Only cluster admins can assign/remove SCC to a service account.
```bash
oc create serviceaccount testsa
oc adm policy add-scc-to-user anyuid -z testsa
```
How to set 
