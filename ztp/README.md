Terminologies

ACM Operator - Operator installation which will provide the crd like multiclusterhub
Multiclusterhub > Should be enabled
Assisted Service - To deploy the openshift clusters from RHACM, it is deployed automatically when we create the multiclusterhub.
Provisioning - To watch all namespaces 
AgentServiceConfig - Here we specify the ISO and RootFS images which are hosted on the mirror registry HTTP server.

We specify below. RootFS and RHCOS URL.

```bash
apiVersion: agent-install.openshift.io/v1beta1
kind: AgentServiceConfig
metadata:
name: agent
spec:
# ...
osImages:
- cpuArchitecture: x86_64
openshiftVersion: "4.18"
rootFSUrl: https://<host>/<path>/rhcos-live-rootfs.x86_64.img
url: https://<host>/<path>/rhcos-live.x86_64.iso
```
Configure cluster to use a disconnected mirror registry by creating a configmap.

apiVersion: v1
kind: ConfigMap
metadata:
name: assisted-installer-mirror-config
namespace: multicluster-engine
labels:
app: assisted-service
data:
ca-bundle.crt: |
-----BEGIN CERTIFICATE-----
<certificate_contents>
-----END CERTIFICATE-----
registries.conf: |
unqualified-search-registries = ["registry.access.redhat.com", "docker.io"]
[[registry]]
prefix = ""
location = "quay.io/example-repository"
mirror-by-digest-only = true
[[registry.mirror]]
location = "mirror1.registry.corp.com:5000/example-repository" 


When we create this configmap then mirrorregistryref is updated in the **AgentServiceConfig** resource

```bash
apiVersion: agent-install.openshift.io/v1beta1
kind: AgentServiceConfig
metadata:
name: agent
namespace: multicluster-engine
spec:
databaseStorage:
volumeName: <db_pv_name>
accessModes:
- ReadWriteOnce
resources:
requests:
storage: <db_storage_size>
filesystemStorage:
volumeName: <fs_pv_name>
accessModes:
- ReadWriteOnce
resources:
requests:
storage: <fs_storage_size>
mirrorRegistryRef:
name: assisted-installer-mirror-config
osImages:
- openshiftVersion: <ocp_version>
```
Configuring hub cluster to use unauthenticated registries.
```bash
oc edit AgentServiceConfig agent
apiVersion: agent-install.openshift.io/v1beta1
kind: AgentServiceConfig
metadata:
name: agent
spec:
unauthenticatedRegistries:
- example.registry.com
- example.registry2.com
```
Configuring hub cluster with ArgoCD.
Siteconfig - RHACM use Siteconfig CRs to generate the day1 managed cluster installation CRs for ArgoCD. Each ArgoCD application can managed 300 Siteconfig CRs
