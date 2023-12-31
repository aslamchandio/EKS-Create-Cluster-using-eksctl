# Create EKS Cluster & Node Groups

## Step-00: Introduction
- Understand about EKS Core Objects
  - Control Plane
  - Worker Nodes & Node Groups
  - Fargate Profiles
  - VPC
- Create EKS Cluster
- Associate EKS Cluster to IAM OIDC Provider
- Create EKS Node Groups
- Verify Cluster, Node Groups, EC2 Instances, IAM Policies and Node Groups

## Prerequisites for EKS Cluster is VPC 

### References (My Git Hub Repo - terraform-on-awscloud-eks/01- VPC-EKS-Private-Prod/)
- https://github.com/aslamchandio/terraform-on-awscloud-eks/tree/main/01-%20VPC-EKS-Private-Prod

## Step-01: Create EKS Cluster using eksctl
- It will take 15 to 20 minutes to create the Cluster Control Plane 
```
# Create Cluster

eksctl create cluster --name=ekscluster1 \
                      --region=us-east-1 \
                      --version=1.27 \
                      --vpc-private-subnets=subnet-068ea6ef94d6b160c,subnet-0f29b5a0aa2442166,subnet-085e8b49791ed3149 \
                      --without-nodegroup 

# Get List of clusters
eksctl get cluster                  
```


## Step-02: Create & Associate IAM OIDC Provider for our EKS Cluster
- To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create &  associate OIDC identity provider.
- To do so using `eksctl` we can use the  below command. 
- Use latest eksctl version (as on today the latest version is `0.21.0`)
```                   
# Template
eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluter-name> \
    --approve

# Replace with region & cluster name
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster ekscluster1 \
    --approve
```



## Step-03: Create EC2 Keypair
- Create a new EC2 Keypair with name as `EKSKey`
- This keypair we will use it when creating the EKS NodeGroup.
- This will help us to login to the EKS Worker Nodes using Terminal.

## Step-04: Create Node Group with additional Add-Ons in Public Subnets
- These add-ons will create the respective IAM policies for us automatically within our Node Group role.
 ```
# Create Public Node Group   
eksctl create nodegroup --cluster=ekscluster1 \
                       --region=us-east-1 \
                       --name=ekscluster1-ng-private \
                       --node-type=t2.medium \
                       --nodes=2 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --subnet-ids=subnet-068ea6ef94d6b160c,subnet-0f29b5a0aa2442166,subnet-085e8b49791ed3149 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=EKSKey \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access \
                       --node-private-networking 


```

## Step-05: Verify Cluster & Nodes

### Verify NodeGroup subnets to confirm EC2 Instances are in Public Subnet
- Verify the node group subnet to ensure it created in public subnets
  - Go to Services -> EKS -> eksdemo -> eksdemo1-ng1-public
  - Click on Associated subnet in **Details** tab
  - Click on **Route Table** Tab.
  - We should see that internet route via Internet Gateway (0.0.0.0/0 -> igw-xxxxxxxx)

### Verify Cluster, NodeGroup in EKS Management Console
- Go to Services -> Elastic Kubernetes Service -> eksdemo1

### List Worker Nodes
```
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=<clusterName>

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# Our kubectl context should be automatically changed to new cluster
kubectl config view --minify
```

### Verify Worker Node IAM Role and list of Policies
- Go to Services -> EC2 -> Worker Nodes
- Click on **IAM Role associated to EC2 Worker Nodes**

### Verify Security Group Associated to Worker Nodes
- Go to Services -> EC2 -> Worker Nodes
- Click on **Security Group** associated to EC2 Instance which contains `remote` in the name.

### Verify CloudFormation Stacks
- Verify Control Plane Stack & Events
- Verify NodeGroup Stack & Events

### Login to Worker Node using Keypai kube-demo
- Login to worker node
```

# For Windows 7
Use putty
```

## Step-06: Update Worker Nodes Security Group to allow all traffic
- We need to allow `All Traffic` on worker node security group

## Additional References
- https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
- https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html


## Step-02: Create EKS Cluster with yml

### Create Fargate Profile manifest
```yml

apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: it-dev-ekscluster1
  region: us-east-1
  version: "1.27"

vpc:
  subnets:
    private:
      us-east-1a: { id: subnet-081aca0fe6b50ca33 }
      us-east-1b: { id: subnet-09d3b735772ad924f }
      us-east-1d: { id: subnet-054d58867d2e4d33c }

  clusterEndpoints:
    publicAccess: true
    privateAccess: true

  publicAccessCIDRs: ["39.51.7.206/32"]

kubernetesNetworkConfig:
  ipFamily: IPv4
  serviceIPv4CIDR: 10.44.0.0/16

iam:
  withOIDC: true  

addons:
- name: vpc-cni # no version is specified so it deploys the default version
  attachPolicyARNs:
    - arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
- name: coredns
  version: latest # auto discovers the latest available
- name: kube-proxy
  version: latest
- name: aws-ebs-csi-driver
  wellKnownPolicies:      # add IAM and service account
    ebsCSIController: true


#nodeGroups: []

managedNodeGroups:
- name: it-dev-ekscluster1-ng-private
  desiredCapacity: 2
  minSize: 2
  maxSize: 4
  instanceType: t2.medium
  volumeSize: 20
  amiFamily: AmazonLinux2
  privateNetworking: true
  subnets:
      - us-east-1a
      - us-east-1b
      - us-east-1d

  ssh: # use existing EC2 key
    publicKeyName: EKSKey
    sourceSecurityGroupIds: ["sg-090f46ad7233fd0f7"]

  iam:
    withAddonPolicies:
        autoScaler: true
        albIngress: true
        appMesh: true
        externalDNS: true  
  

  labels:
    workshop-default: 'yes'

cloudWatch:
  clusterLogging:
    enableTypes: ["api", "audit", "authenticator", "controllerManager", "scheduler"]
    logRetentionInDays: 7               
  
```

## Step-03: Create Fargate Profiles using YAML files
```
# Create Fargate Profiles using YAML file
eksctl create fargateprofile -f 01-eks-cluster.yml
```