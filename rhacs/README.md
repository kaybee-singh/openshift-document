RHACS Components.
```bash
Central
secure cluster
sensor
scanner
collector
```
Upstream project for RHACS is stackrox
RHACS can be deployed in these ways. Recommended method is by using the Operator.
- Operator based installation
- Helm
- With roxctl utility

For central to work in online mode we need the internet connection so that central can connect to definitions.stackrox.io to download the vulnerability definitions.
Scanner component also needs the internet connect. If we can not give the internet access then we can deploy a web proxy for this.
For offline mode we can manually provide all the vulnerability definitions and image details.


RHACS is deployed in the stackrox namespace.
After installation an admin user is created and its password is saved in a secret central-htpasswd in stackrox namespace.

oc get secret central-htpasswd -n stackrox

To change the password of user we can create a new secret and provide in the central configuration.

Architecture:-

Central provides the WebUI, API to show vulnerability, compliance and scan results. It is usually active on one cluster and communicates with other cluster by securedcluster.

Secured cluster should be installed on every cluster that RHACS manages.

RHACS has two main components. Central and Scanner. Scanner is based on stackrox.

