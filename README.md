# DevOps Traning

- My Notes: https://drive.google.com/file/d/1hFz3QV2HNpz1vHo5Pqn4_PsgYXgshfvD/view
- Source Git Repo: https://github.com/lerndevops/labs
- Complete CI-CD flow: https://github.com/lerndevops/labs/blob/master/cicd-flow/CICD-FLOW.jpeg
- Attendance Sheet: https://docs.google.com/spreadsheets/d/1KRz7ldOmXmL9B6ISVDrTNpqsj2OQj9WNx199LvPPpnA/edit#gid=0
- Shared Files/Docs: https://drive.google.com/drive/folders/1ceejSFVRlSAWHc-mq-4GaRCV939cWscU

---

# Material

- https://iomegamct-my.sharepoint.com/personal/ramkumar_iomegamct_onmicrosoft_com/_layouts/15/onedrive.aspx?id=%2Fpersonal%2Framkumar%5Fiomegamct%5Fonmicrosoft%5Fcom%2FDocuments%2FAirbus%20%2D%20Training%20Materials

A. Training Notes
https://iomegamct-my.sharepoint.com/:w:/g/personal/ramkumar_iomegamct_onmicrosoft_com/EX1Ve0VYH7pKqWzxfjPWJr0B95jR40ohE6flfEhS8A5Q0Q?e=2MQwcz

Expires on 31st December 2021
Password: Prestige123$$/?

B. Training Materials
https://iomegamct-my.sharepoint.com/:f:/g/personal/ramkumar_iomegamct_onmicrosoft_com/EpzhL2Ol5IlPn4cOpr1x4gUB0jNJdVm4Jkm0N5vvUcGG1Q?e=OvZ0vd

Expires on 31st December 2021
Password: Prestige123$$/?

C. Code Snippets, Commands and Source Code References
https://codebunk.com/b/7801100385235/

---

# DevOps Traning Creds for Lab

- URL: https://console.aws.amazon.com/
- Account #: 142198642907
- AWS Console Password: Prestige123$$/?
- Username: ABUSER15
- Region: CA-CENTRAL-1

---

# Containerization

- Most of the builds of Jenkins master-slaves are done by K8 we don't manage manually

---

# How do I assign a custom DNS server to an EC2 instance (VPC) which persists across reboots?

- \$ sudo su
- \$ cat /etc/resolv.conf
- \$ nslookup google.com
- \$ cd /etc/dhcp
- \$ ls
- \$ nano dhclient.conf (Add this line at last)
  ```
   supersede domain-name-servers 172.111.94.36;
  ```
- \$ reboot
- \$ cat /etc/resolv.conf
- \$ nslookup google.com (any ping to any website happens via my random DNS name -> 172.111.94.36 )
- Detailed Explaination: https://www.youtube.com/watch?v=0xDYDTss2jQ

---

# NOTE:

- Currently docker-compose.yml is old style of writing docker comands -> Helm Charts is new fashion https://helm.sh/
- Increase SSH Timeout: https://www.tecmint.com/increase-ssh-connection-timeout/
