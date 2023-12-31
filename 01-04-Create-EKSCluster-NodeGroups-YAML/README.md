# Step-01: Create EKS Cluster with yml

## Create Fargate Profile manifest
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

# Step-02: Create Fargate Profiles using YAML files
```
# Create Fargate Profiles using YAML file
eksctl create fargateprofile -f 01-eks-cluster.yml
```