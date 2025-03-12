# **Certified Kubernetes Administrator (CKA) Practice Questions**

## **1. Cluster Architecture, Installation & Configuration**

### **1.1 Check Kubernetes Version**

**Command:**

```sh
kubectl version --short
```

### **1.2 Create a Kubernetes Cluster using Kubeadm**

**Steps:**

1. Install dependencies (`kubeadm`, `kubelet`, `kubectl`)
2. Initialize the control plane:
   ```sh
   kubeadm init --pod-network-cidr=192.168.0.0/16
   ```
3. Configure `kubectl` access:
   ```sh
   mkdir -p $HOME/.kube
   cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   chown $(id -u):$(id -g) $HOME/.kube/config
   ```
4. Install a network plugin (e.g., Calico, Flannel)
5. Join worker nodes using the `kubeadm join` command from step 2.

### **1.3 Upgrade the Cluster from v1.27 to v1.28**

**Steps:**

```sh
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
apt update && apt install -y kubeadm=1.28.0-00
kubeadm upgrade apply v1.28.0
apt install -y kubelet=1.28.0-00 kubectl=1.28.0-00
systemctl restart kubelet
kubectl uncordon <node>
```

### **1.4 Join a Worker Node to the Cluster**

**Command:**

```sh
kubeadm token create --print-join-command
```

Run the output command on the worker node.

### **1.5 Backup & Restore etcd**

**Backup Command:**

```sh
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key
```

**Restore Command:**

```sh
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db
```

---

## **2. Troubleshooting Kubernetes**

### **2.1 Troubleshoot Pods Not Running**

**Check pod status:**
```sh
kubectl get pods -A
```

**Describe the pod for errors:**
```sh
kubectl describe pod <pod-name> -n <namespace>
```

**Check logs for issues:**
```sh
kubectl logs <pod-name> -n <namespace>
```

**Common Fixes:**
- Check `kubectl get events -A` for scheduling failures.
- Restart the pod by deleting it (`kubectl delete pod <pod-name>`).
- Check resource requests/limits (`kubectl describe pod`).

### **2.2 Troubleshoot Node Issues**

**Check node status:**
```sh
kubectl get nodes -o wide
```

**Common Node Issues:**
- `NotReady`: Check `systemctl status kubelet` on the node.
- Disk pressure: Free up space on `/var/lib/kubelet`.

---

## **3. Security & RBAC**

### **3.1 Create a Service Account with Limited Permissions**

**Create a service account:**
```sh
kubectl create serviceaccount test-user
```

**Create a role with limited permissions:**
```sh
kubectl create role test-role --verb=get,list --resource=pods -n default
```

**Bind the role to the service account:**
```sh
kubectl create rolebinding test-binding --role=test-role --serviceaccount=default:test-user -n default
```

---

## **4. Networking & Storage**

### **4.1 Verify CNI Plugin is Working**

**Check network plugin pods:**
```sh
kubectl get pods -n kube-system -l k8s-app=flannel
```

**Common Fixes:**
- Restart network plugin pods if they are failing.

### **4.2 Create a Persistent Volume & Claim**

**Persistent Volume YAML:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

**Persistent Volume Claim YAML:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

**Apply the configurations:**
```sh
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
```

---

## **5. Theoretical Questions & Answers**

### **5.1 What are the main components of the Kubernetes Control Plane?**

- API Server
- Scheduler
- Controller Manager
- etcd (Key-value store)
- Cloud Controller Manager (optional)

### **5.2 How does Kubernetes handle networking?**

- Uses a flat network model where each pod gets a unique IP.
- Supports CNI plugins like Calico, Flannel, Cilium.
- Provides Services (ClusterIP, NodePort, LoadBalancer) for communication.

### **5.3 What are the key differences between Deployments, StatefulSets, and DaemonSets?**

| Feature          | Deployment | StatefulSet | DaemonSet |
|-----------------|------------|------------|------------|
| Pod Identity    | Random | Stable | One per node |
| Scaling        | Stateless | Stateful | Fixed to nodes |
| Use Cases      | Web apps | Databases | Monitoring agents |

---

These additions cover key topics needed for the exam. Let me know if you need further refinements! 🚀


## **6. Additional Topics**  

### **6.1 Pod Security Policies (PSP) & SecurityContext**  
- Kubernetes deprecated PSP; instead, use SecurityContext and Admission Controllers.  
- Example of SecurityContext:  
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: secure-pod
  spec:
    securityContext:
      runAsUser: 1000
      runAsNonRoot: true
    containers:
    - name: nginx
      image: nginx
      securityContext:
        readOnlyRootFilesystem: true
  ```  

### **6.2 Ingress Controller & TLS Configuration**  
- Set up Nginx Ingress Controller.  
- Create an Ingress rule with TLS for HTTPS traffic.  

### **6.3 Custom Resource Definitions (CRDs)**  
- Define a CRD for custom resources in Kubernetes.  
- Example of a CRD:  
  ```yaml
  apiVersion: apiextensions.k8s.io/v1
  kind: CustomResourceDefinition
  metadata:
    name: crd-example
  spec:
    group: example.com
    versions:
    - name: v1
      served: true
      storage: true
    scope: Namespaced
    names:
      plural: examples
      singular: example
      kind: Example
  ```  

### **6.4 Metrics Server & Horizontal Pod Autoscaling (HPA)**  
- Enable Metrics Server for auto-scaling:  
  ```sh
  kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
  ```  
- Create an HPA for a deployment:  
  ```sh
  kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=1 --max=10
  ```
