
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
