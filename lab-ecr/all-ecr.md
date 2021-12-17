# First Case-study for ECR -> Deploying Container image in ECR repo

- Case study Repo: https://github.com/tsabunkar/scbv4-casestudy
- Documentation about this case-study is ./first-case-study-ecr
- Goto EC2
- Launch Instances
  - Amazon Machine Image (AMI): Amazon Linux 2 AMI (HVM) - Kernel 5.10
  - Instance Type: t3 small
  - Add storage: 16GB
  - Add Tags
    - Tag => Name: tejas-docker-server
  - Configure Security Group:
    - Add rule
    - Type: All trafic, Source: Anywhere
  - Launch
  - Make sure you have: tejas-admin-keypair-aws.pem (key-pair)
- In local terminal connect to the above Ec2 instance
- \$ sudo ssh -i "tejas-admin-keypair-aws.pem" ec2-user@ec2-3-237-184-12.compute-1.amazonaws.com
- Now let us install- git, docker, etc
  - \$ sudo yum update
  - \$ sudo yum install -y git
  - \$ sudo amazon-linux-extras install docker
  - \$ sudo service docker start
  - \$ sudo usermod -aG docker ec2-user
  - \$ sudo chmod 666 /var/run/docker.sock
- Check if git, docker installed properly
  - \$ git --version
  - \$ docker --version
  - \$ docker info
  - \$ docker images
- Make practice directory and clone ur git project in this ec2 instance
  - \$ mkdir practices
  - \$ cd practices/
  - Now clone this repo: \$ git clone https://github.com/tsabunkar/scbv4-casestudy
  - \$ ls
  - \$ cd scbv4-casestudy/
  - \$ ls -l
  - \$ cd source/
  - \$ cd calculation-offer-service
  - \$ cd CalculationServiceAPISolution/
  - \$ ls -l
- Now Login to AWS Subscription from EC2 Instance
  - Before that generate IAM role
    - Goto IAM
    - Users (tab)
    - (select user) abuser15
    - Security Credentials (tab)
    - Create access key
    - Download .csv file --> give u : Access key ID, Secret access key
  - Back to local terminal (ec2 instance)
  - \$ aws --version
  - \$ aws configure
  - AWS Access Key ID [None]: AKIAXGBYOXBBZWB (from above generated IAM Role -> devUser1)
  - AWS Secret Access Key [None]: X6QOOZDkhw8hNnSfEal4oTvPm3KMwDZ3BOW
  - Default region name [None]: ca-central-1
  - Default output format [None]: json
- Now let us Create ECR Repository, Since Before building image, We Need to push image to ECR repository
  - Goto ECR in aws
  - Amazon ECR: Repositories
  - Create Repository
    - Visibility settings: Private
    - Repository name: tejas-casestudy-calculation-service
    - Create Repositopry
  - (Select the Repository created) tejas-casestudy-calculation-service
  - View push commands
  - Execute command showed in your local terminal: Push commands for tejas-casestudy-calculation-service (NOTE: in local terminal ur there in path -> /home/ec2-user/practices/scbv4-casestudy/source/calculation-offer-service/CalculationServiceAPISolution)
    - \$ aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 494039644227.dkr.ecr.us-east-1.amazonaws.com
    - \$ docker build -t tejas-casestudy-calculation-service .
    - \$ docker tag tejas-casestudy-calculation-service:latest 494039644227.dkr.ecr.us-east-1.amazonaws.com/tejas-casestudy-calculation-service:latest
    - \$ docker push 494039644227.dkr.ecr.us-east-1.amazonaws.com/tejas-casestudy-calculation-service:latest
    - Goto ECR Repository -> tejas-casestudy-calculation-service
    - Refersh > you would see new image pushed here !!
- Now let us pull monogo db, Attach network
  - \$ docker pull mongo:latest
  - \$ docker images
  - \$ docker network create training-network
  - \$ docker network inspect training-network
  - \$ docker network ls
  - \$ docker run -d -t -p 27017:27017 --name mongo-server --network training-network mongo
  - \$ docker container ls
  - \$ docker inspect 145 (container ID)
  - \$ docker ps
  - \$ docker exec -it 145 bash
  - Inside the mongo-container let us run some mongo commands
    - \$ mongo --host mongo-server --port 27017
    - \$ show dbs
    - \$ use admin
    - \$ show collections
    - \$ exit (come out from mongo shell)
    - \$ exit (come out of devUser1 IAM user)
- \$ sudo yum install nano -y (Run this command terminal)
- Now let us do Basic config of Mongo server and run image
  - \$ nano prod-settings.env -> (edit the MONGO-CONNECTION_STRING value to -> mongodb://mongo-server:27017/calculationservicesrequestsdb)
  ```
  ASPNETCORE_URLS=http://*:80
  ASPNETCORE_ENVIRONMENT=Development
  MONGO_CONNECTION_STRING=mongodb://mongo-server:27017/calculationservicesrequestsdb
  ```
  - \$ cat prod-settings.env
  - Run Docker image SYTNAX -> docker run -d -t -p 8080:80 --env-file=./prod-settings.env --name calculationrestservice --network training-network <your-ECR-repository-name>
  - \$ docker run -d -t -p 8080:80 --env-file=./prod-settings.env --name calculationrestservice --network training-network tejas-casestudy-calculation-service
  - \$ docker container ls
  - \$ docker logs 07cbcbc2142a (container ID of image -> tejas-casestudy-calculation-service)
- open the postman
  - URL: SYNTAX -> http://<public-ipv4-dns-name-of-ec2-instance>:8080/api/calculation-services/process
    - \$ http://ec2-3-97-71-235.ca-central-1.compute.amazonaws.com:8080/api/calculation-services/process
  - Method -> POST
  - Headers
    - Content-Type:application/json
    - Accept:application/json
  - Body -> ./ecr-post.json
  - raw -> json > beautify
  - send
  - Hope you got 200OK
- Now let us check if the posted message is saved in our mongodb which was ecr hosted image
  - Back terminal
  - \$ docker container ls
  - \$ docker exec -it 145270cf2181 bash (image ID -> mongo)
  - Now we are Inside the mongo-container
  - \$ mongo --host mongo-server --port 27017
  - \$ show dbs
  - \$ use calculationservicesRequestsDb
  - \$ show collections
  - \$ db.calculationservicerequests.find()
  - \$ exit
  - \$ exit
- Verify the records are present

---

# Doing above activity for other micro-services

- Goto ECR
- Create Repository
- Repository Name: tejas-email-service
- Create Repository
- Repository Name: tejas-identity-verification-service
- Create Repository
- Repository Name: tejas-creditcard-service
- Create Repository
- Repository Name: tejas-creditcard-identity-verification-response-daemon
- Once all the above repositories are created in ECR, Then go back local terminal where this ec2 instance is running
- \$ cd ~/practices/scbv4-casestudy/
- \$ cd source/
- Let us first target for <creditcard-identity-verification> Microservice
  - \$ cd creditcard-identity-verification-response-daemon/
  - Goto ECR > (click) tejas-creditcard-identity-verification-response-daemon > View push commands
  - \$ docker build -t tejas-creditcard-identity-verification-response-daemon .
  - \$ docker tag tejas-creditcard-identity-verification-response-daemon:latest 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-creditcard-identity-verification-response-daemon:latest
  - \$ docker push 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-creditcard-identity-verification-response-daemon:latest
  - Verify the image is pushed in ecr repo
- Now us do for <creditcard-service> Microservice
  - \$ cd ..
  - \$ cd creditcard-service/
  - Goto ECR > (click) tejas-creditcard-service > View push commands
  - \$ docker build -t tejas-creditcard-service .
  - \$ docker tag tejas-creditcard-service:latest 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-creditcard-service:latest
  - \$ docker push 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-creditcard-service:latest
  - Verify the image is pushed in ecr repo
- Now us do for <email-service> Microservice
  - \$ cd ..
  - \$ cd email-service/
  - \$ docker build -t tejas-email-service .
  - \$ docker tag tejas-email-service:latest 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-email-service:latest
  - \$ docker push 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-email-service:latest
  - Verify the image is pushed in ecr repo
- Now us do for <identity-verification-service> Microservice
  - \$ cd ..
  - \$ cd identity-verification-service/
  - \$ docker build -t tejas-identity-verification-service .
  - \$ docker tag tejas-identity-verification-service:latest 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-identity-verification-service:latest
  - \$ docker push 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-identity-verification-service:latest
- Let us pull Message Broker -> Rabbit MQ
  - \$ docker image ls
  - \$ docker pull rabbitmq:3-management
  - \$ docker image ls
  - \$ docker network ls
  - \$ docker run -d -t -p 5672:5672 -p 15672:15672 --name rabbitmq-server --network training-network rabbitmq:3-management\
  - \$ docker ps --format "table {{.Image}}\t{{.Ports}}\t{{.Names}}"
  - \$ python3 --version
  - \$ curl http://localhost:15672/cli/rabbitmqadmin --output rabbitmqadmin
  - \$ python3 rabbitmqadmin -u guest -p guest -V / -H localhost -P 15672 declare queue name=verification-inputs-queue durable=true
  - \$ python3 rabbitmqadmin -u guest -p guest -V / -H localhost -P 15672 declare queue name=verification-outputs-queue durable=true
  - \$ python3 rabbitmqadmin -u guest -p guest -V / -H localhost -P 15672 declare queue name=email-requests-queue durable=true
- Let us VERIFY RABBITMQ QUEUES ARE CREATED Properly
  - Open the Browser -> http://<dns-ec2-instance>:15672
    - \$ http://ec2-3-97-71-235.ca-central-1.compute.amazonaws.com:15672
    - This will open Rabbit MQ
    - User Name: guest Password: guest
  - Goto Queues (Tab)
  - Make sure that all queues are created like-> email-requests-queue, verification-inputs-queue, verification-outputs-queue
- Now let us do some config
  - \$ cd ..
  - \$ cd calculation-offer-service
  - \$ cd CalculationServiceAPISolution/
  - \$ cat prod-settings.env (ensure -> MONGO_CONNECTION_STRING=mongodb://mongo-server:27017/calculationservicesrequestsdb)
  - \$ docker container rm calculationrestservice -f
  - Let us run <tejas-casestudy-calculation-service>
  - \$ docker image ls
  - \$ docker run -d -t -p 8080:80 --name calculation-server --env-file=./prod-settings.env --network training-network 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-casestudy-calculation-service
- Let us run do some cofig and run <identity-verification-service>
  - \$ docker logs c7
  - \$ cd ..
  - \$ cd ..
  - Goto soruce/identity-verification-service folder
  - \$ cd identity-verification-service/
  - \$ nano prod-settings.env
  ```
  MONGO_CONNECTION_STRING=mongodb://mongo-server:27017/identityverificationrequestsdb
  CLOUDAMQP_URL=amqp://guest:guest@rabbitmq-server
  INPUT_QUEUE=verification-inputs-queue
  RESPONSE_QUEUE=verification-outputs-queue
  ```
  - \$ docker image ls
  - \$ docker run -d -t --env-file=./prod-settings.env --name identity-verification-service --network training-network 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-identity-verification-service
  - \$ docker logs eff
- Let us run do some cofig and run <creditcard-identity-verification-response-daemon>
  - \$ cd ..
  - \$ cd creditcard-identity-verification-response-daemon/
  - \$ nano prod-settings.env
  ```
  CLOUDAMQP_URL=amqp://guest:guest@rabbitmq-server
  MONGO_CONNECTION_STRING=mongodb://mongo-server:27017/creditcardservicesdb
  VERIFICATION_RESPONSE_QUEUE=verification-outputs-queue
  ```
  - \$ docker image ls
  - \$ docker run -d -t --env-file=./prod-settings.env --name identity-verification-response-deamon --network training-network 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-creditcard-identity-verification-response-daemon
  - \$ docker logs 10c
- Let us run do some cofig and run <email-service>
  - \$ cd ..
  - \$ cd email-service/
  - \$ nano prod-settings.env
  ```
  CLOUDAMQP_URL=amqp://guest:guest@rabbitmq-server
  AWS_ACCESS_KEY=
  AWS_ACCESS_SECRET_KEY=
  AWS_REGION=ap-south-1
  MONGO_CONNECTION_STRING=mongodb://mongo-server:27017/emailrequestsdb
  EMAIL_REQUESTS_QUEUE=email-requests-queue
  ```
  - NOTE: Modify the Access ID, AWS_ACCESS_SECRET_KEY & DON't Change the AWS_REGION -> Place where you register the email verification in AWS UI
  - \$ docker image ls
  - \$ docker run -d -t --env-file=./prod-settings.env --name email-service --network training-network 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-email-service
  - \$ docker logs 83c
- Let us run do some cofig and run <creditcard-service>
  - \$ cd ..
  - \$ cd creditcard-service/
  ```
  FLASK_DEBUG=true
  FLASK_ENV=development
  FLASK_RUN_PORT=8080
  FLASK_APP=./api.py
  CLOUDAMQP_URL=amqp://guest:guest@rabbitmq-server
  MONGO_CONNECTION_STRING=mongodb://mongo-server:27017/creditcardservicesdb
  CALCULATION_SERVICE_URL=http://calculation-server:80/api/calculation-services/process
  VERIFICATION_INPUTS_QUEUE=verification-inputs-queue
  EMAIL_REQUESTS_QUEUE=email-requests-queue
  FROM_EMAIL_ADDRESS=tsabunkar@gmail.com
  ```
  - \$ docker image ls
  - \$ docker run -d -t -p 9090:8080 --env-file=./prod-settings.env --name creditcard-service --network training-network 142198642907.dkr.ecr.ca-central-1.amazonaws.com/tejas-creditcard-service
  - \$ docker logs 758
- Now got AWS UI
- Switch ur region to -> AP-SOUTH-1 (bcoz you have given this as AWS_REGION)
- Search : Amazon Simple Email Service
- Configuration (tab) -> Verified identities
- Create Identity
- Identity details: Email Address
- Email address: tsabunkar@gmail.com
- Create
- Now Open your Pernsoal ID Mailbox, click the verification address. Check AWS Console, Your Email Address is verified !!
- Open Postman
  - POST method
  - URL: (sytnax) http://DNS-of-ec2-intsance:9090/
    - http://ec2-3-97-71-235.ca-central-1.compute.amazonaws.com:9090/
  - Content-Type:application/json
  - Accept:application/json
  - Body -> Raw
  - message: ./post-use-study.json
- Now make sure you recieved email in your personal email-id

## Automate above Steps as Stages in Jenkins Pipeline

- Goto EC2
- Launch Instance
- AMI: Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type
- Instance Type: t2.micro (free tier)
- Storage : 16GB
- Add tags -> Name: ec2-jenkins-master
- Configure Security Group
  - Security group name: Tejas-Security-Group
  - Add rule: All Trafic
  - Source: Anywhere
- Review & Launch
- Key-pair -> tejas-admin-keypair-aws | RSA
- Launch Instances
- Open local terminal
- Goto directory where ur keypair -> tejas-admin-keypair-aws is present
- \$ sudo ssh -i "tejas-admin-keypair-aws.pem" ec2-user@ec2-3-84-24-62.compute-1.amazonaws.com
- Let us install Jenkins inside this ec2 instance
  - \$ sudo su -
  - \$ yum update
  - \$ yum install java-1.8.0-openjdk-devel -y
  - \$ java -version
  - \$ javac
  - \$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
  - \$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
  - \$ sudo amazon-linux-extras install epel -y
  - \$ yum install daemonize -y
  - \$ yum install jenkins -y
  - \$ systemctl start jenkins
  - \$ systemctl status jenkins
- Goto Browser: (to configure jenkins ui)
  - Get the Public IPv4 DNS --> From EC2 instance
  - \$ http://<Public IPv4 DNS>:8080
  - \$ http://ec2-3-84-24-62.compute-1.amazonaws.com:8080
  - In local get the adminstrator password for Jenkins UI
    - \$ cat /var/lib/jenkins/secrets/initialAdminPassword ==> faf516d9e2ce44348324905dad470b17
  - Give the above Admininstrator password in Jenins UI > Continue
  - Install Suggested Plugins
  - Getting started:
    - Username: tejas-admin
    - Password: 1234
    - Confirm password: 1234
    - Full name: tejas sabunkar
    - email: tsabunkar@gmail.com
  - save and continue
  - save and finish
  - Jenkins URL : http://ec2-3-84-24-62.compute-1.amazonaws.com:8080/
  - start using jenkins
- Manage Jenkins
- Security > Configure Global Security
- Agents
  - TCP port for inbound agents: Random
  - save
- Manage Jenkins
  - System Configuration > Manage Nodes and Clouds
  - New Node
  - Node Name: Master_EC2_Node
  - (select) Permanent Agent
  - OK
  - Description: Master Node which is running on EC2 instance name: ec2-jenkins-master
  - Remote root directory: /home/ec2-user/jenkins
  - Labels: Master_EC2_Node
  - Usage: Only Build Jobs with Label expressions matching the node
  - Launch Method: Launch agent by connecting it to the controller
  - Save
- Manage Jenkins -> Manage Nodes & Clouds -> Select the Instance: Master_EC2_Node
- Click on Agent Master_EC2_Node
- Download
  - launch agent Launch agent from browser --> jenkins-agent.jnlp
  - Run from agent command line: agent.jar --> agent.jar
- launch fileZilla
- File > site manager
- New site
  - site name: aws-ec2-jenkins-master
  - General (tab)
    - Protocol: SFTP (select)
    - host: ec2-3-84-24-62.compute-1.amazonaws.com
    - port : 22
    - Logon Type: Key file
    - Key file: <Browse for file tejas-admin-keypair-aws.pem>
      - Ensure: Select file type as PEM file
    - User: ec2-user
    - Connect
- Once successfully connected to ec2 instance at right hand side you should see the AWS EC2 instance remote directories :)
- Goto Downloads folder(left side) from local and select -> [jenkins-agent.jnlp, agent.jar] file and drag to right side (/home/ec2-user)
- Now goto local master terminal
- \$ exit (incase you are as root)
- \$ ls -l --> Ensure both jenkins-agent.jnlp, agent.jar files are present
- In the Jenkins UI Master_EC2_Node execute Run from agent command line, with the secret stored in a file: (on ur local terminal)
  - \$ echo d8aeabc902d5193b266e0d7a8eeec77442e60f58464ce27143b5b7196998e5c2 > secret-file
  - \$ java -jar agent.jar -jnlpUrl http://ec2-3-84-24-62.compute-1.amazonaws.com:8080/computer/Master%5FEC2%5FNode/jenkins-agent.jnlp -secret @secret-file -workDir "Master_EC2_Node"
  - Onces it says Connected !! (in terminal)
  - Go back to jenkins UI and referesh the Master_EC2_Node -> you should not see the CROSS icon in front of computer
- Let us create new pipeline
  - Jenkins Dashboard
  - New Item
  - Enter an item name : first-test-pipeline
  - Select: pipeline
    - Script: Jenkinsfile-first-pipeline-script
  - Ok
- Build Now
- ERROR: Error cloning remote repo 'origin'
  hudson.plugins.git.GitException: Could not init /var/lib/jenkins/workspace/first-test-pipeline
  - SOLUTION: That was bcoz in ec2-instance you did not have git installed, Check via command:
    - \$ git --version
    - \$ sudo yum install -y git
- Build History tab, Click build number
- Console output
- Once test build is successful,cheers !!
- let us BlueOcean plugin
  - Dashboard > Manage Jenkins > Manage Plugins
  - Avaliable (tab)
  - Search for: Blue Ocean
  - Select and install without restart
  - Dashboard > Open Blue Ocean
  - Let us add new pipeline from blueocean
    - New Pipeline
    - Where do you store your code?: Github
    - Connect to Github -> Create access token
      - Personal access tokens : My Jenkns PAT
      - Generate token
      - Copy the token --> ghp_lRwFaKqHYPJ66k2HTLofRz8vlO3Lai2ayx
      - paste in ur Connect to Github token
      - Connect
    - Which organization does the repository belong to?: tsabunkar
    - Choose a repository: BlueOcean_Pipeline (select) > create pipeline
    - Completed
    - Build is sucessfull !!
