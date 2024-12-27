# kubeadm-aws -vm setup:

Prequisites: 
1. security group Contorl plane:

rule-1:
type - custom IP 
protocol - TCP
port range - 10248-10260
source - custom IP (172.31.0.0/16)

rule-2:
type - custom IP 
protocol - TCP
port range - 2379- 2380
source - custom IP (172.31.0.0/16)

rule-3:
type - custom IP 
protocol - TCP
port range - 6443
source - custom IP (172.31.0.0/16)

rule-4:
type - SSH
protocol - TCP
port range - 22
source - My IP 

----------------------------------------------------

2. security group worker nodes:

rule-1:
type - HTTP 
protocol - TCP
port range - 80
source - any where

rule-2:
type - custom IP 
protocol - TCP
port range - 10250
source - custom IP (172.31.0.0/16)

rule-3:
type - custom IP 
protocol - TCP
port range - 10256
source - custom IP (172.31.0.0/16)

rule-4:
type - SSH
protocol - TCP
port range - 22
source - My IP 
----------------------------------------------------------------------------------------
Create a IAM Role with below Police: 

IAM Policy for the controller node

-> Create an IAM role with the following permissions and attach it to the controller node
-----------------------------------------------------------------------------------------
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

--------------------------------------------------------------------------------------------------------

IAM Policy for the Worker Nodes

-> Create an IAM role with the following permissions and attach it to the worker nodes.
---------------------------------------------------------------------------------------------------------
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

-------------------------------------------------------------------------------------------------------------------

-> Now Create 3 VM - Instance one for control Plane and rest of them for worker-node with below configuration

master-node:

name - master-node
instance type - t2.medium
AMI - Ubuntu 22.04
generate a .pem key
security Group - select the security group created for control-plane
launch instance

-----------------------------------------------------------------------

worker nodes:

name - worker-1/worker 2
instance type - t2.medium
AMI - Ubuntu 22.04
use the generated pem key
security Group - select the security group created for worker-nodes
launch instance
-------------------------------------------------------------------------------------------------------------------------
Now attach the created master and worker node Roles to  Master and worker node Instances:

process: go to EC2 Console -> select the Instance -> Action -> Security -> Modify the IAM Role and attach it

------------------------------------------------------------------------------------------------------------------------------------------------------------------ 
ADD TAgs To AWS Resources:

If the AWS resources are managed by one cluster, we can provide the tag key kubernetes.io/cluster/kubernetes and the tag value as owned and
if the resources are managed by multiple clusters,the tag key would be the same as kubernetes.io/cluster/kubernetes and the tag value should be mentioned as shared.

NOTE: Tags should be added to resourecs the controller and worker node consumes, such as VPC, Subnet, EC2 instance, Security Group, etc.

Key:  kubernetes.io/cluster/kubernetes
Value: owned 
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
KUBE-ADM SETUP:
----------------------------------------------------------------------------------------------------------------------------------------------------------------

Run the below steps on the Master VM

SSH into the Master EC2 server

Disable Swap using the below commands

swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
Forwarding IPv4 and letting iptables see bridged traffic
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Verify that the br_netfilter, overlay modules are loaded by running the following commands:
lsmod | grep br_netfilter
lsmod | grep overlay

# Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command:
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
Install container runtime
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

# Check that containerd service is up and running
systemctl status containerd
Install runc
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
install cni plugin
curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
Install kubeadm, kubelet and kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet=1.29.6-1.1 kubeadm=1.29.6-1.1 kubectl=1.29.6-1.1 --allow-downgrades --allow-change-held-packages
sudo apt-mark hold kubelet kubeadm kubectl

kubeadm version
kubelet --version
kubectl version --client
Note: The reason we are installing 1.29, so that in one of the later task, we can upgrade the cluster to 1.30

Configure crictl to work with containerd
sudo crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock

initialize control plane
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=172.31.89.68 --node-name master
Note: Copy the copy to the notepad that was generated after the init command completion, we will use that later.

Prepare kubeconfig
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
Install calico
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml

curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml -O

kubectl apply -f custom-resources.yaml
Perform the below steps on both the worker nodes
Perform steps 1-8 on both the nodes
Run the command generated in step 9 on the Master node which is similar to below
sudo kubeadm join 172.31.71.210:6443 --token xxxxx --discovery-token-ca-cert-hash sha256:xxx
If you forgot to copy the command, you can execute below command on master node to generate the join command again
kubeadm token create --print-join-command


