
# Creating Users on Openshift

A brief description of what this project does and who it's for

1. Create htpasswd file with a list of users



2. To deploy this project run

```bash
$ mkdir ~/htconfigure/
$ htpasswd -c -B -b ~/htconfigure/htpasswd admin redhat
$ htpasswd -B -b ~/htconfigure/htpasswd test redhat
$ htpasswd -B -b ~/htconfigure/htpasswd dean redhat
$ htpasswd -B -b ~/htconfigure/htpasswd bob redhat

```

3. Add required roles to the users.


```bash
$ oc adm policy add-cluster-role-to-user cluster-admin admin
$ oc adm policy add-cluster-role-to-user cluster-admin sam
$ oc adm policy add-cluster-role-to-user cluster-admin dean

```

4. Fetch the oauth

```bash
$ oc get oauth cluster -o yaml > ~/htconfigure/oauth.yaml

```

5. Edit spec section and edit it to look like below.


```bash
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: localusers
    mappingMethod: claim
    name: myusers
    type: HTPasswd


```

6. Replace the oauth content.

```bash
$ oc replace -f oauth.yaml

```
## Authors

- [@kaybee-singh](https://www.github.com/kaybee-singh)
