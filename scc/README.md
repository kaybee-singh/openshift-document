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
