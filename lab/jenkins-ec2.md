# CI-CD process using Jenkins in EC2 instance

## Create key-pairs in Ec2

- Goto > EC2
- Network & Security > Key Pairs
- Name: Tejas-ec2keypair, Key pair type: RSA, private key - .pem
- create key pair (download)

## Create new ec2 instance:

- Launch EC2 instancee
- Quick start (tab)
- Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type
- t2 small
- Configure instance details :
  - Auto-assign Public IP- Enable
  - DNS Hostname: Enable resource-based IPv4 (A record) DNS requests
- Add storage
  - Size: 16 GB
- Add tags
  - key:vale ==> Name: Tejas-Jenkins-Master
- Configure Security Group
  - Security group name: Tejas-Security-Group
  - Add rule: All Trafic
  - Source: Anywhere
- Review & Launch
- Select an exisiting key pair > Launch Instance
- In local:
  - Goto Downloads folder (where you have downloaded .pem file)
  - \$ ssh -i "<.pem file-name>" ec2-user@public-ipv4-dns-name
  - \$ sudo ssh -i "Tejas-ec2keypair.pem" ec2-user@ec2-3-98-121-42.ca-central-1.compute.amazonaws.com
- Connect to instance (can be followed from)
  - Ec2 > instances > i-0845581b29a1e55ee > Connect to instance
  - SSH Client (tab)
  - \$ chmod 400 Tejas-ec2keypair.pem
  - \$ ssh -i "Tejas-ec2keypair.pem" ec2-user@ec2-3-98-121-42.ca-central-1.compute.amazonaws.com

## Start local EC2 Instance in personal laptop and install,start Jenkins in Master Ec2 Instance

- Once logged to ec2 instance using ssh client from local
- \$ sudo su -
- \$ yum update
- \$ yum install java-1.8.0-openjdk-devel
- \$ java -version
- \$ javac
- \$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
- \$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
- \$ sudo amazon-linux-extras install epel
- \$ yum install daemonize -y
- \$ yum install jenkins -y
- \$ systemctl start jenkins
- \$ systemctl status jenkins

- Goto Browser: (to configure jenkins ui)
  - Get the Public IPv4 DNS --> From EC2 instance
  - \$ http://<Public IPv4 DNS>:8080
  - \$ http://ec2-3-98-121-42.ca-central-1.compute.amazonaws.com:8080
  - In local get the adminstrator password for Jenkins UI
    - \$ cat /var/lib/jenkins/secrets/initialAdminPassword ==> faf516d9e2ce44348324905dad470b17
  - Give the above Admininstrator password in Jenins UI > Continue
  - Install Suggested Plugins
  - Getting started:
    - Username: tejas-admin
    - Password: 1234
    - Confirm password: Tejas
    - Full name: tsabunkar@gmail.com
  - save and continue
  - save and finish
  - start using jenkins
- NOTE: Don't terminate the ec2 instance, just stop instance (this will not delete the setup)
  - Instance state: Stop Instance
  - Stop

## Adding slaves to Jenkins master

## Adding plugins
