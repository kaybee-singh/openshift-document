1. Ingress certificate is used by the Openshift router (ingress controller) to secure incoming HTTPS traffic on the Openshift routes.
2. By Default Openshift use a self-signed certificate for the ingress controller.
3. Whenever a user created an openshift route then it will use the ingress certificate for SSL.
4. That route further routes the traffic to the particular service.
5. A custom ingress certificate can be specified in ingresscontroller resource in Openshift.
6. This resource is created with default name in openshift-ingress-operator namespace.
7. When we add the custom certificate in ingresscontroller resource then router pods are restarted in the openshift-ingress namespace.

```bash
oc get pods -n openshift-ingress         
NAME                              READY   STATUS    RESTARTS   AGE
router-default-7748f4cbf8-24m4f   1/1     Running   0          7d21h
router-default-7748f4cbf8-gf5hm   1/1     Running   0          7d21h
```
8. Openshift operator pod resides in following namespace.
```bash
oc get pods -n openshift-ingress-operator
NAME                               READY   STATUS    RESTARTS        AGE
ingress-operator-fcc7d5dfd-4qpgp   2/2     Running   5 (6d19h ago)   7d21h
```
