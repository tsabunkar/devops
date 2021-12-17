# Ansible

- Ec2
- Launch instance
- AMI: Redhat -> Red Hat Enterprise Linux 8 (HVM), SSD Volume Type
- t3.small
- Number of instances -> 3
- Default storage -> 10GB
- Add rule:
  - HTTP
- Launch
- Rename the ec2 instances -> Tejas-Ansbile-Master, Tejas-Node1, Tejas-Node2
- Connect to all the three instance locally
- In master and node server
  - \$ sudo su -
  - \$ yum update
- In master node
  - \$ yum install ansible -y
  - \$ ansible --version
  - useradd ansibleadmin
  - passwd ansibleadmin (tejas1234)
- In Node server
  - \$ sudo useradd ansibleadmin
  - \$ sudo passwd ansibleadmin
- In Both master/nodes
  - \$ yum install nano -y
  - \$ nano /etc/sudoers (goto last line) (add below line)
  ```
  ec2-user        ALL=(ALL)       NOPASSWD: ALL
  ansibleadmin    ALL=(ALL)       NOPASSWD: ALL
  ```
  - \$ nano /etc/ssh/sshd_config (change thePasswordAuthentication from no to yes `PasswordAuthentication yes`)
  - \$ service sshd restart
  - \$ service sshd status
  - \$ exit
  - \$ exit
- from all the ec2 instance master/nodes, exit Completely ...
- now login to master and nodes using ansibleadmin user
  - \$ ssh ansibleadmin@dns-name
  - \$ sudo ssh ansibleadmin@ec2-3-98-37-84.ca-central-1.compute.amazonaws.com (Master)
  - \$ sudo ssh ansibleadmin@ec2-15-222-23-225.ca-central-1.compute.amazonaws.com (node1)
  - \$ sudo ssh ansibleadmin@ec2-15-222-23-225.ca-central-1.compute.amazonaws.com (node2)
- Master should be able to access node1 and node2 without password (use key-gen technqiue)
  - In master node
    - ssh-keygen (press enter 3 times)
    - Get private Ip Address of Node1 from ec2
      - Node1: 172.31.47.161
      - Node2: 172.31.33.108
    - ssh-copy-id <private_ip_add>
      - \$ ssh-copy-id 172.31.47.161
      - \$ ssh-copy-id 172.31.33.108
    - ssh <private_ip_add> (check/verify)
      - \$ ssh 172.31.47.161
      - \$ ssh 172.31.33.108
    - Adding hosts in ansible
      - \$ sudo nano /etc/ansible/hosts
      - Go to last add Both ip address of Node1 and Node2
      ```
      [productionservers]
      172.31.47.161
      172.31.33.108
      ```
      - \$ ansible all --list
      - \$ ansible productionservers --list
      - \$ ansible productionservers -m ping
    - \$ cat > index.html
    ```
    <h1> Welcome to the World of Ansible!!! </h1>
    ```
    - \$ cat index.html
    - \$ ansible productionservers -m copy -a "src=./index.html dest=/home/ansibleadmin"
    - \$ ansible productionservers -m yum -a "name=httpd state=latest" --become
    - \$ ansible productionservers -m service -a "name=httpd state=started" --become
    - Open Browser
      - browser -> http://<public-dns-node-1>/
      - NODE-1: http://ec2-15-222-23-225.ca-central-1.compute.amazonaws.com/
      - NODE-2: http://ec2-15-222-24-151.ca-central-1.compute.amazonaws.com/

# Playbook

- In master Node
  - \$ nano playbook1.yaml (copy content from ./playbook1.yaml)
  - Passing information securly b/w master to node
    - \$ nano secrets.yaml (content from ./secrets.yaml)
    - \$ cat secrets.yaml
    - \$ ansible-vault encrypt secrets.yaml (tejas1234)
    - \$ cat secrets.yaml
    - \$ nano playbook2.yaml (copy content from ./playbook2.yaml)
    - \$ ansible-playbook playbook2.yaml --ask-vault-pass

# Deploying webapp into the docker container using Ansible

- \$ nano playbook3.yaml (copy content from ./playbook3.yaml)
- \$ ansible-playbook playbook3.yaml --syntax-check
- \$ ansible-playbook playbook3.yaml
- Goto Browser either of the node should server you this static appln
  - http://ec2-15-222-23-225.ca-central-1.compute.amazonaws.com/
  - http://ec2-15-222-24-151.ca-central-1.compute.amazonaws.com/

# install below Nodes

- Install AWS CLI
  - \$ uname -m (x86_64)
  - \$ sudo dnf install zip (install zip in centos)
  - \$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  - \$ unzip awscliv2.zip
  - \$ sudo ./aws/install
  - \$ aws --version
- Install <aws-iam-authenticator>
  - \$ curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator
  - \$ chmod +x ./aws-iam-authenticator
  - \$ mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
  - \$ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
  - \$ aws-iam-authenticator help
- Install <kubectl> in AWS Linux ec2 machine
  - \$ curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
  - \$ chmod +x ./kubectl
  - \$ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
  - \$ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
  - \$ kubectl version --short --client
- Install <terraform>
  - Visit -> https://learn.hashicorp.com/tutorials/terraform/install-cli
  - Goto Centos/RHEL Tab
  - \$ sudo yum install -y yum-utils
  - \$ sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
  - \$ sudo yum -y install terraform
  - \$ terraform -version
  - \$ touch ~/.bashrc
  - \$ terraform -install-autocomplete
- Install <git>
  - \$ yum install git
  - \$ git --version
