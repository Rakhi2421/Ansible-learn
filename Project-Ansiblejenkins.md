## Project-Ansible Integration in jenkins

Demo Overview:
1) Create a do server for jenkins.
2) Create a dedicated server for ansible.
3) Install ansible on that server
4) Execute Ansible Playbook from jenkins file to configure 2 EC2 Instances.

Tasks:
- Create 2 EC2 Instances
- Configure everything from scratch with ansible
- Create a pipeline in jenkins
- Connect pipeline to java maven project
- Creates Jenkins file that executes ansible playbook on the remote ansible server.

Step 1: Create an instance and consider it as Ansible server Control node
- Install Ansible   - apt install ansible
- Install boto3     - apt install python3-pip, pip3 install boto3 botocore
- Install botocore
- Configure AWS Credentials
  - mkdir .aws  at home
  - cd .aws/
  - vim credentials
Step2: Create 2 EC2 instances in AWS
Step 3: Jenkins file: Copy jenkins file from Jenkins to Ansible server
