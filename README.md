# Ansible-learn

## Module-Overview

- Core Concepts:
1. Ansible Inventory
2. Ad-Hoc Commands
3. Tasks,Plays & Playbooks
4. Ansible Config file

- Working with Common Modules
1. Server Configuration
2. Configuring Applications
3. Working with files

- Ansible Collections & Galaxy
- Using Variables
- During demos we'll see:
  - Troubleshooting
  - Conditionals
  - Privilege Escalation
  - and more!
- Advanced Topics & Integrations with other Technologies:
  - Dynamic Incentory for EC2
  - Ansible & Terraform
  - Ansible & Docker
  - Ansible & Kubernetes
  - Ansible & Jenkins Pipeline
  - Ansible Roles

# Introduction to Ansible...

## What is Ansible?
- It is a tool to automate IT tasks.
- It supports all infrastructure from Operating systems to cloud providers.
- Ansible is agentless
- Ansible uses YAML 

## Why use Ansible?
Ansible helps us in 4 different ways.
- Execute tasks from your own machine. Instead of SSH into all remote servers.
- Configuration/Installation/Deployment steps in a single Yaml file. Instead of doing it manually or using a couple of shell scripts
- Re-use same file multiple times and for different environments.
- Humans are prone to errors, More reliable and less likely for errors.

## How Ansible Works?
- Ansible works with modules.
  Modules:
 - Modules are basically small programs that do the actual work.
 - Modules are get pushed to the target servers by control machine.
 - Modules do their work and get removed.
 - Modules are very granular
 - One Module can do one small specific task.

# Ansible Playbook

```bash
- hosts: databases                        ---> Where should these tasks execute? - Hosts                          |            |
  remote_user: root                       ---> With which user should these tasks execute? - Remote_user          |            |
  vars:                                   |                                                                       |            |
    tablename: foo                        | ---> Use variables for repeating values                               |            |
    tableowner: someuser                  |                                                                       |            |
                                                                                                                  |            |
tasks:                                                                                                            |            |
  - name: Rename table {{ tablename }} to bar             ---->  Description of task  |                           |            |
    postgresql_table:                         ---->  Module Name          |                                       | ---> play  |
      table: {{ tablename }}                             |                            | --->  task                |   for      |
      rename: bar                            | ---> Arguments             |                                       |    database|
   - name: Set owner to some user            |                                                                    |            |
     Postgresql_table:                       |                                                                    |            |
        name: foo                            | --> task                                                           |            |----> Playbook
        owner: someuser                      |                                                                    |            |
                                                                                                                               |
- hosts: webserver                                                                    |                                        |
  remote_user: root                                                                   |                                        |
                                                                                      |                                        |  
tasks:                                                                                |                                        |
  - name: Rename table foo to bar                                                     |---> play   for webserver               |
    postgresql_table:                                                                 |                                        |
      table: foo                                                                      |                                        |
      rename: bar                                                                     |                                        |


```
# Ansible Installation

We can install ansible on your local linux system or on a remote server.  
Control node:
- The machine that runs ansible
- Windows isn't supported for the control node.
- and manages target servers

# Setup Managed server
- In target servers, we need to make sure python3 is installed.

# Ansible Inventory
Ansible inventory file
- File containing the data about the ansible client servers.
- "hosts" meaning the managed servers
- default location for file: /etc/ansible/hosts


Grouping Hosts
- you can put each host in more than one group
- you can create groups that track:
  - Where - a datacenter/region,e.g. east,west
  - WHAT - e.g. database servers, web servers etc.
  - WHEN - which stage, e.g. dev, test, prod environment

Ansible ad-hoc Commands
-   ad-hoc commands are not stored for future uses
-   a fast way to interact with the desired servers.
```bash
   $: ansible [pattern] -m [module] -a "[module options]"

e.g. ansible all -i hosts -m ping
```
- [pattern] = targeting hosts and groups
   - "all" = default group, which contaions every host
- [module] = discrete units of code

Hosts file
```bash
$: vi hosts
[droplet]
134.2.2.2  ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=root
134.2.2.3  ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=root

or

$: vi hosts

[droplet]
134.2.2.2  
134.2.2.3

[droplet:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_user=root
```

## Add EC2 Instances to Inventory

```bash
$: vi hosts

[droplet]
134.2.2.2  
134.2.2.3

[droplet:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_user=root

[aws]
134.2.2.2  
134.2.2.3 ansible_python_interpreter=/usr/bin/python3

[aws:vars]
ansible_ssh_private_key_file=~/Downloads/ansible.pem
ansible_user=ec2-user
```

# Managing Host Key Checking

Host Key checking
- It is enabled by default in ansible
- it guards against server spoofing and man-in-the-middle attacks

## Authorized keys & Known hosts

There are 2 ways to work with host jey checking.
1. Add host server in known hosts file.
2. disable host key checking

# Add host server in known hosts file.
1. Add a host server key to ansible server known hosts file using the below cmd.  
$: ssh-keyscan -H 165.22.201.197 >> ~/.ssh/known_hosts  
2. Add a Ansible server public key to remote server .ssh/authorized_keys using the below cmd.   
$: ssh-copy-id root@188.160.30.219

# Disable Host Key Checking
This method is less secure but is used for Ephemeral Infrastructure.  
Ephemeral Infrastructure
- Servers are dynamically created an deleted.

```bash

$: vim ~/.ansible.cfg

[defaults]
host_key_checking = False
```

# Config file default locations
- /etc/ansible/ansible.cfg
- ~/.ansible.cfg

# Intro to Ansible Playbook

Ansible Playbooks - Infrastructure as code
- Ansible Configuration files are treated like code
- Ansible is idempotency
  - most Ansible modules check whether the desired state has already been achieved.
  - they exit without performing any actions

Playbooks
- ordered list of tasks
- Plays & tasks run in order from top to bottom.
- written in yaml
- A playbook cabn have multiple plays
- A Play is a group of tasks

"Gather Facts" Module
- it is automatically called by playbooks
- to gather useful variables about remote hosts that can be used in playbooks.
- So Ansible provides many facts about the system, automatically.

Demo-Project
- Create Ansible project
- save in source control, like git
2 Minimum required
  1. the managed nodes to target.
  2. atleast one task to execute.
 

```bash
$: mkdir Ansible
$: cd Ansible
$: vi hosts
[webserver]
134.2.2.2  
134.2.2.3

[webserver:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_user=root

$: vi my-playbook.yaml
---
-name: Configure Nginx web server
 hosts: webserver
 tasks:
 - name: install nginx server
   apt:
     name: nginx
     state: latest
# to install specific version
     name: nginx=1.18*
     state: present
 - name: start nginx server
   service:
     name: nginx
     state: started

# For every project we have to create a sepearte ansible.cfg file
$:vi ansible.cfg
[defaults]
host_key_checking = False

# Execute Playbook
$: ansible-playbook -i hosts my-playbook.yaml

### To Uninstall a Package

$: vi my-playbook.yaml
---
-name: Configure Nginx web server
 hosts: webserver
 tasks:
 - name: uninstall nginx server
   apt:
     name: nginx
     state: absent
 - name: stop nginx server
   service:
     name: nginx
     state: stopped

# For every project we have to create a sepearte ansible.cfg file
$:vi ansible.cfg
[defaults]
host_key_checking = False

# Execute Playbook
$: ansible-playbook -i hosts my-playbook.yaml

```

## Modules Overview


   




