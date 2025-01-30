### Method 1 - To pull images from a private registry in a particular namespace, follow below steps.

1. Login on your private registry

```bash
podman login quay.io --authfile authfile
cat  authfile
{
	"auths": {
		"quay.io": {
			"auth": "token-dssfsfdfhjk="
		}
	}
}     
```
2. Create secret in the same namespace
```bash
oc project target-namespace
oc create secret generic quay-pull \                                                                                                                                         ok 
    --from-file=.dockerconfigjson=authfile \
    --type=kubernetes.io/dockerconfigjson
oc get secret quay-pull
```
3. Now link the secret with default service account in the same namespace.
```bash
 oc secret link default quay-pull --for=pull
```
4. Now try to create a pod from the private registry image.
```bash
oc create deploy image-test --image=docker.io/user/image
```

### Method 2- To configure image pull from a private registry on the cluster level by using the global pull secret, follow below steps.

1. Extract the global pull secret and save in a file.
```bash
oc get secret/pull-secret -n openshift-config --template='{{index .data ".dockerconfigjson" | base64decode}}' > authfile
```
2. Login to the registry and provide the file where you want to store the token.
```bash
oc registry login --registry=quay.io --auth-basic=username:password --to=authfile
```
3. Now set the data in the global pull secret.
```bash
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=authfile
```
