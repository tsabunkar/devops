# Terraform

- \$ uname -m (x86_64 ==> Then its 64bit arch)
- https://www.terraform.io/downloads.html
  - Extract the terraform file
  - Once extracted in download folder
  - mv ~/Downloads/terraform /usr/local/bin/ (Move this file to local/bin)
  - \$ terraform -help
- HashiCorp Terrform install VSCode extension
- create file -> versions.tf, outputs.tf, etc
- \$ git clone https://github.com/tsabunkar/terraform-practice1.git
- \$ cd terraform-practice1
- get the AMI ID of your region and ubuntu 20.04 ...
  - Goto EC2 Instances
  - ca-central-1 region
  - search for ubuntu -> There you get AMI ID for ur region -> ami-0bb84e7329f4fa1f7 (Change this value variables.tf file in terraform-practice1 repo)
- create key pair
  - EC2
  - Key Pairs (under Network & Security)
  - Create Key pair
  - Name: tejas-ec2
  - Create Key pair -> this will download `tejas-ec2.pem`
- \$ terraform init (inside `/home/tejas/Downloads/terraform-practice1`)
- \$ terraform plan -out "plan1"
- \$ terraform apply "plan1"
- \$ terraform show
- Goto ec2 instance you should see this ec2 would have spinned now
- \$ terraform destroy

# Leverage Modularity of Program in Terraform

- \$ git clone https://github.com/tsabunkar/terraform-practice2.git (Donwload folder)
- \$ cd terraform-practice2
- Change the env variables as per your creds in -> terraform.tfvars
- \$ terraform init
- \$ terraform plan -out "plan1"
- \$ terraform apply "plan1"
- \$ terraform show
- Check this url
  - http://dns-name:9090
  - http://ec2-3-99-49-220.ca-central-1.compute.amazonaws.com:9090/
- NOTE: If you change the `version.tf` file then you need to run `terraform init` command

# EKS in Terraform

- \$ git clone https://github.com/tsabunkar/terraform-practice3.git
- \$ cd terraform-practice3
- \$ terraform init
- \$ terraform plan -out "plan1"
- \$ terraform apply "plan1"
- \$ terraform show

# Helm Charts

- \$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
- \$ chmod 700 get_helm.sh
- \$ sudo ./get_helm.sh
- \$ helm version
- \$ helm create simplechart (say in download folder)
- \$ cd simplechart
- \$ code .
- \$ aws eks update-kube-config --name <name-cluster> --region <region-name>
- after this command only, LENS and KUBECTL shall start working ...
- \$ kubectl create namespace eks-training
- \$ cd ..
- \$ helm install --namespace eks-training eks-training simplechart/ --values simplechart\values.yaml
- \$ kubectl get all -n eks-training
- \$ helm list --all-namespaces
- \$ curl -X GET http://xxxxx-1783597452.ap-south-1.elb.amazonaws.com/
- \$ helm uninstall --namespace eks-training eks-training
- \$ kubectl get all -n eks-training
