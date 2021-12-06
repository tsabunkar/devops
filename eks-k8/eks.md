# Manually provision/configuring EKS Cluster with a Production Ready VPC Settings

- NOTE: We should not manually configure EKS Cluster in Production environment , we should always use -> Teraform, Cloudformation, EKSctl, etc (Preffer teraform, bcoz its cloud augnotic)

## Create EC2 Key-pair (Required if you would like to connect Worker Nodes using SSH)

- Goto EC2
- Network & Security > Key pairs
- Create key pair
  - Name: tejas-key-pair
  - (Download) tejas-key-pair.pem
- (Allow to access to worker node thus we create keypair)
- Create IAM Roles for EKS Cluster and EKS Worker Nodes
  - Goto IAM
  - Roles
  - Create Role
  - AWS service
  - (select) EKS > EKS-Cluster
  - Permission
  - Role name: tejas-eks-cluster-role
    - Create role
  - (now create role for worker nodes, to pull ecr images)
  - IAM
  - Roles
  - Create Role
  - AWS Service
  - EC2
  - Permission:
    - AmazonEKSWorkerNodePolicy
    - AmazonEKS_CNI_Policy
    - AmazonEC2ContainerRegistryReadOnly
  - Role Name: tejas-ec2-worker-node-role
    - create role

## Create VPC Cloud & Enable DNS Host Names in the VPC

- VPC
- your vpc
- create VPC
- name: tejas-vpc
- IPv4 CIDR: 10.24.0.0/16
- create vpc
- Acions > Edit DNS hostname
  - DNS hostnames: (check) Enable
  - Save changes

## Create Subnets & 2/3 Subnets for Private Subnet & 2/3 Subnets for Public Subnet

- let us create 6 subents with below norms
  - tejas-ekspvcprivate1a 1A 10.24.1.0/24
  - tejas-ekspvcprivate1b 1B 10.24.2.0/24
  - tejas-ekspvcprivate1c 1C 10.24.3.0/24
  - tejas-ekspvcpublic1a 1A 10.24.10.0/24
  - tejas-ekspvcpublic1b 1B 10.24.20.0/24
  - tejas-ekspvcpublic1c 1C 10.24.30.0/24
- VPC
  - Subents
  - create subnet
  - NOTE: Make sure all youre subnets are created in different location
  - VPC ID: (Select) tejas-vpc
  - Subnet 1 of 1
    - Subnet name: tejas-ekspvcprivate1a
    - IPv4 CIDR block: 10.24.1.0/24
  - (click) Add new subnet
  - Subnet 2 of 2
    - Subnet name: tejas-ekspvcprivate1b
    - IPv4 CIDR block: 10.24.2.0/24
  - (click) Add new subnet
  - Subnet 3 of 3
    - Subnet name: tejas-ekspvcprivate1c
    - IPv4 CIDR block: 10.24.3.0/24
  - (click) Add new subnet
  - Subnet 4 of 4
    - Subnet name: tejas-ekspvcpublic1a
    - IPv4 CIDR block: 10.24.10.0/24
  - (click) Add new subnet
  - Subnet 5 of 5
    - Subnet name: tejas-ekspvcpublic1b
    - IPv4 CIDR block: 10.24.20.0/24
  - (click) Add new subnet
  - Subnet 6 of 6
    - Subnet name: tejas-ekspvcpublic1c
    - IPv4 CIDR block: 10.24.30.0/24
  - (click) Add new subnet
  - create subnet
- select only public subnet from above (tejas-ekspvcpublic1a, tejas-ekspvcpublic1b, tejas-ekspvcpublic1c)
- (select) tejas-ekspvcpublic1a
  - Actions
  - edit subnet settings
  - (check) Enable auto-assign public IPv4 address
  - Save
- (select) tejas-ekspvcpublic1b
  - Actions
  - edit subnet settings
  - (check) Enable auto-assign public IPv4 address
  - Save
- (select) tejas-ekspvcpublic1c
  - Actions
  - edit subnet settings
  - (check) Enable auto-assign public IPv4 address
  - Save

## Enable Public IPv4 Addressing for Public Subnets & Create Internet Gateway and Associate to the VPC & Create Route Table for Public Subnets, Associate Public Subnets & Define a Route to use I-Gateway for Internet Access

- VPC
- internet gateways (tab)
- create internet gateway
- Name tag: tejas-internetgateway
- Create internet gateway
- actions > Attach to vpc
- Available VPCs -> (search) tejas-vpc
- Attach internet gateway
- Route tables (tab)
- create route table
  - name: tejas-routetable
  - VPC: (select) tejas-vpc
  - create route table
- subnet associations (tab)
  - edit subnet associations
  - (select) public subnets -> (tejas-ekspvcpublic1a, tejas-ekspvcpublic1b, tejas-ekspvcpublic1c)
  - Save associations
- Actions
- Edit routes
- add route
- Destination: 0.0.0.0/0, Target: internet gateway
- save

## Create a NAT Gateway (Make sure that a public subnet is selected through which all communications to the internet happens) & Create a Route Table for Private Subnets and Associate Private Subnets & Define a Route to use NAT Gateway for restricted | filtered internet access

- NAT gateways
- Create Nat gateway
  - Name: tejas-gateway
  - sunet tejas-ekspvcpublic1a
  - Allocate Elastic Ip (click)
  - Create NAT gateway
- Route tables (tab)
  - create route table
  - name: tejas-routetable
  - vpc: tejas-vpc
  - create route table
- subnet associations (tab for that route tables)
  - edit subnet associations
  - select only private -> (tejas-ekspvcprivatec1a, tejas-ekspvcprivatec1b, tejas-ekspvcprivatec1c)
  - save associations
- Actions
- edit routes
- Add route
- Destination: 0.0.0.0/0, Target: nat gateway
- Save changes

## Provision EKS Cluster

- Goto EKS
- Amazon EKS > Clusters
- Add cluster > Create
- Cluster Configuration
  - Name: tejas-eks-cluster
  - Cluster Service Role: tejas-eks-cluster-role
  - Next
- Specify networking
  - VPC: (select) tejas-vpc
  - (ensure all your public and private subnet are selected in Subnets)
  - Security groups -> (select) default
  - Cluster endpoint access -> public and private
  - next
- configure logging
- api server: Enabled
- Enabled all
- next
- create cluster

## Provision EKS Node Groups

- EKS Cluster you can see
- Configuration
- Compute
- Node Groups
- Add group
  - Name: tejas-eks-worker-node
  - select role: worker-node-role
  - next
- Capcity type: On-Demand
- Disk size: 30GB
- max size: 5
- Max unavaliable
  - value: 50 %
  - Next
- Specific networking
  - Subnets: (Select only private, not public)
  - Configure SSH access to nodes
  - (select) your key-pair
  - next
- create
- (Now Download kubectl, AWS CLI)
- (once installed)
- kubectl version --client
- aws configure
- aws update-kubeconfig --name
