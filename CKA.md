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

## **2. Workloads & Scheduling**
### **2.1 Deploy an Nginx Pod on Worker-1**
**YAML File:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
spec:
  nodeSelector:
    kubernetes.io/hostname: worker-1
  containers:
  - name: nginx
    image: nginx
```
**Apply Command:**
```sh
kubectl apply -f nginx-pod.yaml
```

### **2.2 Scale a Deployment**
**Command:**
```sh
kubectl scale deployment my-app --replicas=5
```

### **2.3 Taint a Node**
**Command:**
```sh
kubectl taint nodes worker-1 key=value:NoSchedule
```

### **2.4 Create a Pod with an Environment Variable**
**YAML File:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    env:
    - name: DB_HOST
      value: "postgres"
```

### **2.5 Set CPU & Memory Requests/Limits**
**YAML File:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: busybox
    image: busybox
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

---

## **3. Services & Networking**
### **3.1 Create a ClusterIP Service**
**YAML File:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

### **3.2 Expose a Pod Externally using NodePort**
**Command:**
```sh
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service
```

### **3.3 Troubleshoot Pod Communication Issues**
**Commands:**
```sh
kubectl get pods -o wide
kubectl get networkpolicy
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
```

---

## **4. Storage**
### **4.1 Create a Persistent Volume (PV)**
**YAML File:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

### **4.2 Create a Persistent Volume Claim (PVC)**
**YAML File:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

---

## **5. Troubleshooting**
### **5.1 Debug a Pod Stuck in CrashLoopBackOff**
**Commands:**
```sh
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
```

### **5.2 Check Logs for a Specific Container**
**Command:**
```sh
kubectl logs <pod-name> -c <container-name>
```

### **5.3 Restart a Control Plane Component**
**Command:**
```sh
systemctl restart kubelet
```

---

## **Final Tips**
✅ Practice on a live cluster using `minikube` or `kind`  
✅ Use `kubectl explain <resource>` to understand object specifications  
✅ Set up a hands-on lab environment using `kubeadm` or a cloud-based Kubernetes service  

---

This document covers essential **CKA** topics, including YAML examples and troubleshooting commands. Happy learning! 🚀

