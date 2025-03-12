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

# 6. Theoretical Knowledge Questions

### 6.1 Kubernetes Basics

What are the key components of the Kubernetes control plane?Answer: The control plane includes kube-apiserver, kube-controller-manager, kube-scheduler, etcd, and cloud-controller-manager (if applicable).

What is the difference between a Deployment and a StatefulSet?Answer: A Deployment is used for stateless applications and manages replicas, whereas a StatefulSet is used for stateful applications and ensures unique, stable network identities for each pod.

How does a ReplicaSet ensure high availability?Answer: It ensures the desired number of pod replicas are always running by creating new pods if any fail.

What is the difference between kubectl apply and kubectl create?Answer: kubectl apply is used for declarative updates, while kubectl create is used for imperative creation of resources.

### 6.2 Cluster Management

Explain the role of kube-apiserver, kube-scheduler, and kube-controller-manager.Answer: kube-apiserver handles API requests, kube-scheduler assigns pods to nodes, and kube-controller-manager runs controllers to maintain cluster state.

What happens during a kubeadm init command?Answer: It initializes the control plane, generates certificates, starts control plane components, and sets up the cluster configuration.

How does Kubernetes handle leader election for HA control plane setups?Answer: Kubernetes uses etcd and leader election algorithms in kube-controller-manager to ensure only one leader is active at a time.

What is the function of kubelet and kube-proxy?Answer: kubelet runs on nodes to ensure containers are running, while kube-proxy manages network rules for service communication.

### 6.3 Networking

What are the different types of Kubernetes services?Answer: ClusterIP, NodePort, LoadBalancer, and ExternalName.

How does Kubernetes manage DNS resolution internally?Answer: Kubernetes uses CoreDNS to provide internal DNS resolution for services and pods.

What is a NetworkPolicy, and how does it work?Answer: A NetworkPolicy controls pod communication using rules based on labels, namespaces, and IP blocks.

What is the role of coredns in a Kubernetes cluster?Answer: It provides DNS resolution for service discovery within the cluster.

### 6.4 Storage & Volumes

What are the differences between Persistent Volumes (PVs) and Persistent Volume Claims (PVCs)?Answer: PVs are storage resources in a cluster, while PVCs are requests for storage by applications.

How does dynamic volume provisioning work in Kubernetes?Answer: Kubernetes automatically provisions storage volumes based on PVC requests using storage classes.

What are the different access modes for Persistent Volumes?Answer: ReadWriteOnce (RWO), ReadOnlyMany (ROX), ReadWriteMany (RWX).

### 6.5 Security & Troubleshooting

What are Kubernetes Role-Based Access Control (RBAC) roles?Answer: RBAC roles define permissions for users and groups to access resources in a Kubernetes cluster.

How can you restrict a pod from running as a root user?Answer: Use PodSecurityPolicies or SecurityContext with runAsNonRoot: true.

How do you troubleshoot a pod that is stuck in ContainerCreating state?Answer: Check pod events (kubectl describe pod), storage issues, or node status.

What are taints and tolerations, and how do they affect scheduling?Answer: Taints prevent pods from running on a node unless a toleration allows it.


This document covers essential CKA topics, including YAML examples, troubleshooting commands, and theoretical knowledge. Happy learning! 🚀



