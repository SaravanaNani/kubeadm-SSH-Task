
# Installing Kubernetes on EC2 Instances (Self-Managed)

## Prerequisites

- AWS EC2 instances for Controller (Master) and Worker nodes.
- IAM roles attached with the necessary permissions for both controller and worker nodes.
- Disable source/destination check for the VMs.
- Ensure necessary security group rules are configured.

## Security Group Configuration

### Control Plane Security Group

- **Rule 1**: Type: Custom IP, Protocol: TCP, Port Range: 10248-10260, Source: Custom IP (172.31.0.0/16)
- **Rule 2**: Type: Custom IP, Protocol: TCP, Port Range: 2379-2380, Source: Custom IP (172.31.0.0/16)
- **Rule 3**: Type: Custom IP, Protocol: TCP, Port Range: 6443, Source: Custom IP (172.31.0.0/16)
- **Rule 4**: Type: SSH, Protocol: TCP, Port Range: 22, Source: My IP

> **Note**: Replace `(172.31.0.0/16)` with your VPC CIDR.

### Worker Node Security Group

- **Rule 1**: Type: HTTP, Protocol: TCP, Port Range: 80, Source: Anywhere
- **Rule 2**: Type: Custom IP, Protocol: TCP, Port Range: 10250, Source: Custom IP (172.31.0.0/16)
- **Rule 3**: Type: Custom IP, Protocol: TCP, Port Range: 10256, Source: Custom IP (172.31.0.0/16)
- **Rule 4**: Type: SSH, Protocol: TCP, Port Range: 22, Source: My IP
- **Rule 5**: Type: HTTP, Protocol: TCP, Port Range: 80, Source: 0.0.0.0/0

## IAM Role Configuration

Create and attach IAM roles to the controller and worker nodes with the necessary permissions for the cloud controller manager to interact with the AWS APIs.

# IAM Roles for Controller and Worker Nodes

In order to enable proper interaction between the cloud controller manager and AWS APIs, both the controller and worker nodes require IAM roles with specific permissions. Below are the IAM policies for both the controller and worker nodes.

## IAM Role for Controller Node

The following IAM role should be attached to the controller node to allow it to interact with the AWS APIs and perform necessary operations like scaling, load balancing, and managing EC2 instances.

### IAM Policy for Controller Node

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "ec2:DescribeInstances",
                "ec2:DescribeRegions",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVolumes",
                "ec2:DescribeAvailabilityZones",
                "ec2:CreateSecurityGroup",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:ModifyInstanceAttribute",
                "ec2:ModifyVolume",
                "ec2:AttachVolume",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:CreateRoute",
                "ec2:DeleteRoute",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteVolume",
                "ec2:DetachVolume",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:DescribeVpcs",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:AttachLoadBalancerToSubnets",
                "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:CreateLoadBalancerPolicy",
                "elasticloadbalancing:CreateLoadBalancerListeners",
                "elasticloadbalancing:ConfigureHealthCheck",
                "elasticloadbalancing:DeleteLoadBalancer",
                "elasticloadbalancing:DeleteLoadBalancerListeners",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeLoadBalancerAttributes",
                "elasticloadbalancing:DetachLoadBalancerFromSubnets",
                "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                "elasticloadbalancing:ModifyLoadBalancerAttributes",
                "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:CreateTargetGroup",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:DeleteTargetGroup",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeLoadBalancerPolicies",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeTargetHealth",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:ModifyTargetGroup",
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:DeregisterTargets",
                "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
                "iam:CreateServiceLinkedRole",
                "kms:DescribeKey"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}

```
Instructions:

1. Create a new IAM role in AWS with the above permissions.

2. ttach this IAM role to your controller node to enable it to interact with AWS APIs.

# IAM Role for Worker Node

-> The following IAM role should be attached to the worker node to allow it to pull images from Amazon ECR and interact with EC2 instances.

### IAM Policy for Worker Node
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeRegions",
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:BatchGetImage"
            ],
            "Resource": "*"
        }
    ]
}
```
Conclusion:
Ensure that the IAM roles with the specified permissions are created and correctly attached to the controller and worker nodes. These roles are essential for enabling the Kubernetes components to interact seamlessly with AWS services such as EC2, ECR, and ELB, ensuring smooth operation and scalability of the cluster.

## Step 3: Tag AWS Resources

Tagging AWS resources is essential to configure the Cloud Controller Manager (CCM) in your Kubernetes cluster. By tagging resources, you can ensure that AWS resources, such as EC2 instances, VPCs, Subnets, and Security Groups, are properly associated with the correct cluster. This ensures that the right resources are cleaned up when the cluster is destroyed, preventing unintended changes to other resources.

### Why Tag AWS Resources?

For example, if your cluster uses an AWS Network Load Balancer (NLB) and the cluster is being destroyed, the NLB will be tagged and subsequently destroyed, avoiding any impact on other resources.

### Finding the Cluster ID

To tag AWS resources correctly, you will need the **Cluster ID**. To find your Cluster ID, use the following command:

```bash
kubectl config view
```
The output will look something like this:
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.31.21.29:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```
## Tagging AWS Resources

Once you have the Cluster ID, you can tag the resources associated with your Kubernetes cluster to indicate ownership or shared usage.

### 1. Single Cluster Management

If a resource is only used by one cluster, you should tag it with the following key-value pair:

- **Tag Key:** `kubernetes.io/cluster/<Cluster-ID>`
- **Tag Value:** `owned`

For example, if your Cluster ID is `kubernetes`, the tag would be:

- **Tag Key:** `kubernetes.io/cluster/kubernetes`
- **Tag Value:** `owned`

### 2. Multiple Clusters Sharing Resources

If a resource is shared across multiple clusters, use the following tag:

- **Tag Key:** `kubernetes.io/cluster/<Cluster-ID>`
- **Tag Value:** `shared`

For example, if the resource is shared among multiple clusters, you would use:

- **Tag Key:** `kubernetes.io/cluster/kubernetes`
- **Tag Value:** `shared`

### Resources to Tag

Ensure that you add tags to the following AWS resources that are used by both the controller and worker nodes:

- **VPC**
- **Subnet**
- **EC2 Instances**
- **Security Groups**
- **Elastic Load Balancers (NLB/ALB)**
- **Elastic Block Store (EBS) Volumes**
- **Elastic IPs (EIP)**

By tagging these resources appropriately, you ensure that they are correctly managed and cleaned up when the associated cluster is terminated or modified.

## Conclusion

Properly tagging AWS resources is critical to the efficient operation and management of your Kubernetes cluster. By using the `kubernetes.io/cluster/<Cluster-ID>` tag with either the `owned` or `shared` value, you can effectively manage the resources associated with your cluster and ensure that only the necessary resources are deleted or modified when required.

###

## Steps on the Master Node

### 1. SSH into the Master EC2 instance.
### 2. Disable Swap:

```bash
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#/g' /etc/fstab
```

### 3. Enable IPv4 forwarding:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

### 4. Apply sysctl parameters:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```

### 5. Verify Modules:

```bash
lsmod | grep br_netfilter
lsmod | grep overlay
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

### 6. Install container runtime:

```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

### 7. Install `runc`:

```bash
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

### 8. Install CNI plugin:

```bash
curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
```

### 9. Install kubeadm, kubelet, and kubectl:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.29.6-1.1 kubeadm=1.29.6-1.1 kubectl=1.29.6-1.1 --allow-downgrades --allow-change-held-packages
sudo apt-mark hold kubelet kubeadm kubectl
```

### 10. Initialize the control plane:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=172.31.89.68 --node-name master
```

### 11. Prepare kubeconfig:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 12. Install Calico:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml -O
kubectl apply -f custom-resources.yaml
```

## Steps on Worker Nodes

Perform steps 1-8 from the Master node setup on each worker node.

### 1. Join Worker Node to Cluster:

Run the join command from the Master node:

```bash
sudo kubeadm join 172.31.71.210:6443 --token xxxxx --discovery-token-ca-cert-hash sha256:xxx
```

### 2. Validate Cluster:

```bash
kubectl get nodes
kubectl get pods -A
```

### 3. If Calico Pods are not Healthy:

- Disable source/destination checks on master and worker nodes.
- Update security group rules for TCP port 179 (BIDIRECTIONAL) on both master and worker nodes.
- Update the Calico DaemonSet:

```bash
kubectl set env daemonset/calico-node -n calico-system IP_AUTODETECTION_METHOD=interface=ens5
```

> **Note**: Replace `ens5` with your default interface.

### 4. Workaround for Calico Issues:

If needed, install Calico CNI addon using the manifest:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

This will deploy an older version of Calico (v3.25).

## Conclusion

After completing the steps, your Kubernetes cluster should be up and running with all nodes in the Ready state. Verify by running:

```bash
kubectl get nodes
kubectl get pods -A
```
###

# Configure the Cloud Controller Manager

To configure the AWS Cloud Controller Manager (CCM) for your Kubernetes cluster, follow the steps below.

### 1. Clone the AWS Cloud Controller Repository

Clone the AWS Cloud Provider repository to the controller plane node where you have `kubectl` access.

```bash
git clone https://github.com/kubernetes/cloud-provider-aws.git
```
2. Navigate to the Base Directory
Once the repository is cloned, navigate to the base directory.
This directory contains all the Kubernetes manifests for the cloud controller manager and the Kustomize file.

```bash
cd cloud-provider-aws/examples/existing-cluster/base
```
3. Create the DaemonSet
Create the daemonset using the following command. The -k flag is used for Kustomize to apply the resources.

```
kubectl create -k .
```
4. Verify the DaemonSet is Running
To ensure the daemonset is running properly, use the following command:

```
kubectl get pods -n kube-system
```
By following these steps, you will configure the Cloud Controller Manager to manage AWS resources in your Kubernetes cluster.

###Configuring Kubernetes Components to Use the External Cloud Provider

To integrate Kubernetes with an external cloud provider, such as AWS, you need to configure both the control plane (master) and worker nodes to recognize and interact with the cloud services.

# Control Plane (Master) Node Configuration

1. Kube-API-Server
Locate the kube-apiserver manifest: Typically found at /etc/kubernetes/manifests/kube-apiserver.yaml.

Modify the manifest: Add the --cloud-provider=external flag to the command section:

```
command:
  - kube-apiserver
  - --cloud-provider=external
  - -- other flags
```
Save the changes: The Kubernetes static pod manager will automatically restart the API server with the new configuration.

2. Kube-Controller-Manager
Locate the kube-controller-manager manifest: Typically found at /etc/kubernetes/manifests/kube-controller-manager.yaml.

Modify the manifest: Add the --cloud-provider=external flag to the command section:

```
command:
  - kube-controller-manager
  - --cloud-provider=external
  - --other flags
```
Save the changes: The static pod manager will automatically restart the controller manager with the new configuration.

3. Kube-Scheduler
Locate the kube-scheduler manifest: Typically found at /etc/kubernetes/manifests/kube-scheduler.yaml.

Modify the manifest: Add the --cloud-provider=external flag to the command section:
```
command:
  - kube-scheduler
  - --cloud-provider=external
  - -- other flags
```
Save the changes: The static pod manager will automatically restart the scheduler with the new configuration.

4. Kubelet
Locate the Kubelet configuration file: Typically found at /var/lib/kubelet/config.yaml.

Modify the configuration: Add the cloudProvider: external line under the appropriate section:

```
kind: KubeletConfiguration
cloudProvider: external
# ... other configurations
```
Save the changes: Ensure that this line is added under the appropriate section in the YAML file, maintaining proper indentation.

# Alternatively, if using a systemd environment:

Edit the Kubelet service environment file: Often located at /etc/systemd/system/kubelet.service.d/10-kubeadm.conf.

Modify the environment variable: Add the --cloud-provider=external flag to the KUBELET_EXTRA_ARGS environment variable:

```
KUBELET_EXTRA_ARGS=--cloud-provider=external
```
Reload the systemd daemon and restart the Kubelet service:
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```


### Worker Node Configuration

1. Kubelet

Locate the Kubelet configuration file: Typically found at /var/lib/kubelet/config.yaml.

Modify the configuration: Add the cloudProvider: external line under the appropriate section:

```
kind: KubeletConfiguration
cloudProvider: external
# ... other configurations
```
Save the changes: Ensure that this line is added under the appropriate section in the YAML file, maintaining proper indentation.

# Alternatively, if using a systemd environment:

Edit the Kubelet service environment file: Often located at /etc/systemd/system/kubelet.service.d/10-kubeadm.conf.

Modify the environment variable: Add the --cloud-provider=external flag to the KUBELET_EXTRA_ARGS environment variable:

```
KUBELET_EXTRA_ARGS=--cloud-provider=external
```
Reload the systemd daemon and restart the Kubelet service:
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
By configuring both the control plane and worker nodes with the --cloud-provider=external flag, Kubernetes will interact with the specified external cloud provider, enabling features like load balancing, storage provisioning, and more.

Note: Ensure that all nodes in your cluster are consistently configured to use the external cloud provider to avoid any discrepancies in behavior.

To ensure the Cloud Conroller manager daemonset is running properly, use the following command:

```
kubectl get pods -n kube-system
```
### Provision of Load Balancer and Deployment of Java-app

step-1 create a deployment.yaml file:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: java-app
  template:
    metadata:
      labels:
        app: java-app
    spec:
      containers:
      - name: java-app
        image: saravana2002/java-app:latest
        ports:
        - containerPort: 8080
```
Run the below command to create deployment and verify:

```
kubectl -f apply deployment.yaml
kubeclt get deployments
kubectl get pods -o wide
```
step-2 create a hpa.yaml file for HroizontalPadAutoscaler

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: java-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: java-app-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Run the below command to create Auto-scaling and verify:

```
kubectl -f apply hpa.yaml
kubeclt get hpa -o wide
```
Step-3 create service.yaml file for LoadBalancer service:
```
apiVersion: v1
kind: Service
metadata:
  name: java-app-service
spec:
  selector:
    app: java-app # Ensure this label matches your pod configuration
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80       # Port exposed externally
      targetPort: 8080 # Port your application is listening on
```
Run the below command to create service and verify:

```
kubectl -f apply service.yaml
kubeclt get svc -o wide
```

Now copy the dns from the output of the command ```kubeclt get svc -o wide``` and paste it in the browser to expose your java-app

