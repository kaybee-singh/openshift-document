## This document will cover following sections.

What is a kubeconfig?
Different sections of kubeconfig
What are contexts in kubeconfig?
How to manage contexts?
Safety Measures for kubeconfig
---
1. Kubeconfig file default location for Both Openshift and Kubenetes. If we try to run any kubectl or oc command it will try to find out the kubeconfig file from following location.
~/.kube/config
2. If file is missing then kubectl commands will fail with below error.
```bash
kubectl get nodes
E0808 17:30:54.630140    7763 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
E0808 17:30:54.631113    7763 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
```
3. In case of missing file oc will fail with below error
```bash
oc get nodes
error: Missing or incomplete configuration info.  Please point to an existing, complete config file:
  1. Via the command-line flag --kubeconfig
  2. Via the KUBECONFIG environment variable
  3. In your home directory as ~/.kube/config
To view or setup config directly use the 'config' command.
```
4. We can specify a new custom file with oc. It will create a new config file.
```bash
oc get nodes --kubeconfig=newconfig
```
5. Following are the sections in the kubeconfig.
 - cluster
 - contexts
 - users
 - current-context
6. Below is a sample file
```bash
apiVersion: v1
kind: Config
current-context: minikube
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://api.example.com:6443
  name: api-example.com:6443
contexts:
- context:
    cluster: api.example.com:6443
    user: alice/api.example.com:6443
  name: /api.example.com:6443/alice
- user:
   - name: alice/api.example.com:6443
     user:
      token: sha256~t33OKj6jPQRuQ-LS59HFF9wIdwO0gg20Q

