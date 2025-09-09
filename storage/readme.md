# Kubernetes & OpenShift: Deployments, Config, Secrets, and Storage (Cheat Sheet)

> Practical notes for production-ready Deployments and persistent storage on OpenShift/Kubernetes.

---

## 1) Production‑Grade Deployments

- Avoid shipping raw `oc create deployment ...` to **production**.
- **Generate YAML and customize** before applying:
  - Environment variables & configuration via **ConfigMaps**
  - **Secrets** for sensitive data
  - Resource **requests/limits**, **liveness/readiness** probes
  - Security context, service account, tolerations/affinity, HPA, replicas
- Quick start:
  ```bash
  oc create deployment my-app --image=quay.io/org/app:latest     --dry-run=client -o yaml > deployment.yaml
  ```

---

## 2) ConfigMaps vs Secrets

**ConfigMaps** – non‑sensitive configuration (key/values, files).  
**Secrets** – sensitive values (passwords, tokens, SSH keys, TLS).

Attach either to **Pods/Deployments** as:
- **Environment** variables (`env`, `envFrom`)
- **Volumes** (files mounted into the container)

> Secrets are stored **base64‑encoded** (not encrypted). Enable **encryption at rest** in the cluster and restrict access via **RBAC**.

### Common Secret Types
- **Opaque** (generic key/values)
- **Service account token**
- **Basic auth** (username/password)
- **SSH keys**
- **TLS** (cert/key pair)
- **Docker registry** credentials

### Handy Commands
```bash
# ConfigMap (from literal / file)
oc create configmap app-config   --from-literal=MODE=prod   --from-file=app.yaml

# Secrets (generic / docker-registry / tls)
oc create secret generic db-secrets --from-literal=DB_PASSWORD='p@ss'
oc create secret docker-registry regcred   --docker-server=registry.example.com   --docker-username=user --docker-password=pass
oc create secret tls web-tls --cert=cert.crt --key=cert.key
```

### Mount Examples (YAML)
```yaml
# As environment (from ConfigMap + Secret)
containers:
- name: app
  image: quay.io/org/app:latest
  envFrom:
  - configMapRef: { name: app-config }
  - secretRef:    { name: db-secrets }
```

```yaml
# As files (volumes)
volumes:
- name: cm-vol
  configMap: { name: app-config }
- name: secret-vol
  secret:    { secretName: db-secrets }
containers:
- name: app
  volumeMounts:
  - { name: cm-vol,    mountPath: /etc/app/config }
  - { name: secret-vol, mountPath: /etc/app/secret, readOnly: true }
```

---

## 3) Ephemeral vs Persistent Storage

- **Ephemeral** storage (e.g., `emptyDir`) lives only for the **Pod lifecycle** and **cannot be shared** across Pods. Data is lost on Pod restart/replacement.
- Use **PersistentVolumes (PV)** + **PersistentVolumeClaims (PVC)** for data that must persist.
- Provisioning models:
  - **Static** – admin pre‑creates PVs
  - **Dynamic** – PVC requests create PVs automatically via a **StorageClass** (recommended)

---

## 4) PVC (PersistentVolumeClaim) – What & How

A **PVC** is a **request for storage** by a workload. You specify:
- **Size**, e.g., `20Gi`
- **Access mode**: `RWO`, `RWX`, `ROX`
- **storageClassName** (e.g., ODF/Ceph classes, cloud classes)

**Scope & Permissions**
- **PVC** → **namespaced** (created inside a Project/Namespace)
- **StorageClass** → **cluster‑scoped**
- **PV** → **cluster‑scoped**
- Regular developers typically **cannot** create PV/StorageClass → use **dynamic provisioning**

### Access Modes: What they mean
- **RWO (ReadWriteOnce)** – mounted read‑write by **one node** at a time  
  _Typical backends_: Ceph **RBD**, AWS **EBS**, GCE **PD**, Azure **Disk**, Local PV
- **RWX (ReadWriteMany)** – mounted read‑write by **multiple nodes**  
  _Typical backends_: **NFS**, **CephFS**, Amazon **EFS**, Azure **Files**
- **ROX (ReadOnlyMany)** – mounted read‑only by multiple nodes

> Note: **ext4/xfs** are **filesystems**, not access modes. The sharing behavior is controlled by the **storage backend** and the selected **access mode**.

### PVC Examples
```yaml
# RWO (Ceph RBD / Cloud Disk)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
  namespace: myproj
spec:
  accessModes: ["ReadWriteOnce"]
  resources: { requests: { storage: 20Gi } }
  storageClassName: ocs-storagecluster-ceph-rbd
```

```yaml
# RWX (CephFS / NFS-like shared)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-data
  namespace: myproj
spec:
  accessModes: ["ReadWriteMany"]
  resources: { requests: { storage: 20Gi } }
  storageClassName: ocs-storagecluster-cephfs
```

---

## 5) StorageClass – Picking the Right Class

A **StorageClass** defines the **provisioner** and parameters for dynamic PVs.

**Examples**
- **OpenShift Data Foundation (ODF)** (uses **Ceph** under the hood):
  - **RWO**: `ocs-storagecluster-ceph-rbd` (Ceph **RBD** block)
  - **RWX**: `ocs-storagecluster-cephfs` (Ceph **FS**)
- **Cloud**:
  - AWS: `gp3` (EBS), `efs` (RWX)
  - Azure: `managed-premium` / `premium-rwo` (Disk), `azurefiles` (RWX)
  - GCP: `pd-ssd` (PD), Filestore classes for RWX
- You may also run **upstream Ceph** (open‑source SDS) without ODF.

**Choose on**: cost, performance (IOPS/throughput/latency), replication & reliability, access mode needs.

---

## 6) PersistentVolume (PV) Essentials

- **Access Modes**: `RWO`, `ROX`, `RWX`
- **Volume Modes**: `Filesystem` (default) or `Block` (raw device)
- **Reclaim Policy**: `Delete` or `Retain`
- **Binding Mode** (StorageClass): `Immediate` vs `WaitForFirstConsumer`

---

## 7) Common Volume Types (When to Use)

- **configMap** – mount configuration as files/env (not for persistence)
- **emptyDir** – scratch/temp per‑Pod (ephemeral)
- **hostPath** – path on node (ties Pod to node; avoid in prod; requires elevated perms)
- **local** – Local PVs on node disks (fast, node‑affine; app‑specific suitability)
- **iSCSI** – block storage over IP (requires initiator/CHAP setup)
- **NFS** – simple shared filesystem; fine for non‑critical workloads or read‑heavy content (not ideal for high‑write DBs)

---

## 8) Quick Checklist (for slides)

- **Deployment YAML**: env/config, Secrets, limits, probes, security
- **Config vs Secret**: non‑sensitive vs sensitive; mount as env/files
- **Secrets**: base64‑encoded; encrypt at rest + RBAC
- **Data**: ephemeral vs persistent; PV/PVC + StorageClass
- **PVC**: size + access mode + storageClassName (namespaced)
- **PV/StorageClass**: cluster‑scoped; dynamic provisioning preferred
- **ODF/Ceph**: RBD (RWO), CephFS (RWX); cloud equivalents exist

---

## 9) Useful One‑liners

```bash
# See StorageClasses
oc get storageclass

# Create a PVC quickly (kubectl/oc)
oc apply -f pvc.yaml

# Watch PVC/PV binding
oc get pvc,pv -n myproj -w

# Describe a bound PV
oc describe pv <pv-name>
```

---

### Attribution
Prepared for internal training/handouts. Suitable for OpenShift and upstream Kubernetes.
