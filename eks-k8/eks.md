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

---

# Creating EKS cluster using Automation -> Cloudformation (above steps) ==> Privisioning EKS cluster using eksctl

- IAM
- Users
- get the csv file of ur user
- In local terminal
- aws configure -> enter Access key Id, Screte Access key, region name, Default output format [json]: yaml etc
  - MAKE SURE output format : YAML
- \$ kubectl version --client
- Create new eks cluster using yml file
- Download eksctl from url:
  - https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
  - \$ `curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp`
  - \$ sudo mv /tmp/eksctl /usr/local/bin
  - \$ eksctl version
- create a file named "eksctl-cluster.yaml" and paste the following content
  - NOTE: EKSCTL documentation: https://eksctl.io/
  - \$ touch eksctl-cluster.yaml
  - \$ nano eksctl-cluster.yaml (copy file form ./eksctl-cluster.yaml)
  - \$ cat eksctl-cluster.yaml
- \$ eksctl create cluster -f eksctl-cluster.yaml
- in aws console gui, search for Cloudformation in appropriate region ... You should see the all the resource eks cluster would be created automatically - NO need to manually do vpc, subnets, etc...
- got lense
  - cluster
  - In command prompt local
    - \$ aws eks update-kubeconfig --name tejas-cluster --region ca-central-1
  - Clusters
  - connect

## Install and Provision Metrics Server and Dashboard

- Yml file to get metrics (in local)
  - \$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
  - \$ kubectl get pods -n kube-system
  - \$ kubectl top nodes -n kube-system
- Create admin-service account YAML file
  - \$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
  - \$ kubectl get all -n kubernetes-dashboard
  - Now create a file named "eks-admin-account.yaml" and paste the following content from "./eks-admin-account.yaml"
    - \$ touch eks-admin-account.yaml
    - \$ nano eks-admin-account.yaml
    - \$ kubectl apply -f "eks-admin-account.yaml"
  - Now let us login to k8 dashboard
  - \$ kubectl -n kubernetes-dashboard get secret
  - (identify the user name which is created recently)
  - \$ kubectl -n kubernetes-dashboard describe secret <full-user-name>
  - \$ kubectl -n kubernetes-dashboard describe secret admin-user-token-rwzxn
  - Copy the TOKEN, As it would be used later !!
  - \$ kubectl proxy
  - OPen browser:
    - `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/node?namespace=_all`
    - (select) token
    - Enter token value from above
    - Sign in

---

# Jenins Pipeline -> Read from Git, Build image and push to ECR

- goto ec2
- Start ec2 instance where Jenkins master was running
- local terminal connect to this ec2 instance
  - \$ ssh -i "Tejas-ec2keypair.pem" ec2-user@ec2-35-183-118-133.ca-central-1.compute.amazonaws.com
  - \$ sudo amazon-linux-extras install docker
  - \$ sudo service docker start
  - \$ sudo usermod -aG docker ec2-user
  - \$ sudo chmod 666 /var/run/docker.sock
  - \$ docker images
  - \$ docker info
  - \$ docker ps
  - \$ sudo systemctl restart jenkins
  - \$ sudo systemctl status jenkins
- Login to Jenkins
  - \$ http://ec2-35-183-118-133.ca-central-1.compute.amazonaws.com:8080/
  - \$ uname: tejas-admin, password: you know
  - Upadate Jenkins Configuration
    - Manage Jenkins -> Configure System -> Jenkins Location -> Jenkins URL (update the jenkins url to newly ipv4 dns name that was created: http://ec2-35-183-118-133.ca-central-1.compute.amazonaws.com:8080/)
    - Save
- Goto ECR
  - Create repository
  - Private repo
  - Repository name: tejasv2-casestudy-calculation-service
  - Create repository
- select the repo created now
- View Push commands
- Open a new local Terminal
  - \$ aws configure (abuser15.csv file)
  - \$ Default output format [yaml]: json
  - \$ aws ecr get-login-password --region ca-central-1 (Copy the password in notepad, will be used later)
- Comeback to Jenkins (Now let us create Env Key:value variable for ecr uname and password)
  - Jenkins Dashboard -> Manage Jenkins -> Manage Credentials -> Stores scoped to Jenkins (global) -> Add Credentials
  - Username: AWS
  - Password: (paste the password from -> aws ecr get-login-password COMMAND)
  - ID: ecr-credentials
  - Ok
- Goto github Repo: https://github.com/tsabunkar/airbus-casestudy
- Goback to Jenkins
- Open Blue Ocean
- New pipeline
  - Where do you store your code?: Github
  - Choose a repository: airbus-casestudy
  - Create pipeline
- In Blueocean UI let us create stages
- Click Start (let us define environment variable)
- Environment (click `+` button)
  - NOTE: Remeber in ecr repo you have opened (view push commands) -> Push commands for tejasv2-casestudy-calculation-service
  - Name: ECR_ID, Value: 142198642907.dkr.ecr.ca-central-1.amazonaws.com
  - Name: CALCULATION_SERVICE_IMAGE, Value: tejasv2-casestudy-calculation-service
- Create Stage
- Stage Name: docker-cmds
- Add step: (select) Change current directory
- Path: `source/calculation-offer-service/CalculationServiceAPISolution`
- Child steps
  - Add step: (select) Shell Script
  - \$ pwd (paste this command in text box)
  - Save (by committing to main branch)
- Once build is successfully :)
- let us Add new steps [PLEASE NOTE: Add steps under the above path only] (All shell scripts only)
  - \$ docker build -t $CALCULATION_SERVICE_IMAGE:latest -t $CALCULATION_SERVICE_IMAGE:$BUILD_NUMBER . (paste this command in text box)
  - Save > Commit & Run
  - \$ docker tag $CALCULATION_SERVICE_IMAGE:latest $ECR_ID/$CALCULATION_SERVICE_IMAGE:latest
  - Save > Commit & Run
  - \$ docker tag $CALCULATION_SERVICE_IMAGE:$BUILD_NUMBER $ECR_ID/$CALCULATION_SERVICE_IMAGE:$BUILD_NUMBER
  - Save > Commit & Run
  - Goto git repo under Jenkins file -> https://github.com/tsabunkar/airbus-casestudy/blob/main/Jenkinsfile
    - edit file: (Jenkinsfile)
    ```
    ECR_CREDENTIALS = credentials('ecr-credentials')
    ```
    - Commit changes
  - Goback to Blue Ocean UI, Continue adding steps
  - \$ docker login --username $ECR_CREDENTIALS_USR --password $ECR_CREDENTIALS_PSW $ECR_ID
  - Save > Commit & Run
  - \$ docker image prune -f (delete unwanted images like- <NONE>)
  - Save > Commit & Run
  - \$ docker push $ECR_ID/$CALCULATION_SERVICE_IMAGE:latest (this would push this image to ecr repo)
  - Save > Commit & Run
- Now Verify the pipeline is working ... and make sure that ECR gets the repository ...
- goto ecr repo
- (select) tejasv2-casestudy-calculation-service
- Ensure new latest image is pushed here :)
