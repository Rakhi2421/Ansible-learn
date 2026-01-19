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
Modules
- ALso reffered to as "task plugins"
- Ansible executes a module usually on remote server.
- And collects return values.

## Ansible Collections
### what is a collection?
- A packaging format for bundling and distributing ansible content.
- Can be released and installed independent of other collections.
- All modules are part of collection.
## Ansible plugins
- Pieces of code that add to ansible's functionality or modules.
- you can also write own plugins.
## Ansible Galaxy
-Where we can store our collections and it's like a collection hub.

##Create Own Collection
- For bigger ansible projects
- Collections follow a simple data structure
  - Required:
     a galaxy.yml file (containing metadata) at the root level of collection.
```bash

    collection/
├── docs/
├── galaxy.yml
├── meta/
│   └── runtime.yml
├── plugins/
│   ├── modules/
│   │   └── module1.py
│   ├── inventory/
│   └── ...
├── README.md
├── roles/
│   ├── role1/
│   ├── role2/
│   └── ...
├── playbooks/
│   ├── files/
│   ├── vars/
│   ├── templates/
│   └── tasks/
└── tests/
```   

## Variables in ansible

## Naming of Variables:

- Not Valid: Python Keywords, such as sync Playbook keywords, such as environment.
- Valid: letters, numbers and underscores
- should always start with a letter
- DON't: linux-name, linux name, linux.name or 12 

#### Registered Variables
- Create variables from the output of an ansible task
- this variable can be used in any later tasks in your play.
e.g.
```bash
- name: Ensure app is running
  shell: ps aux | grep node
  register: app_status
- debug: msg={{app_status.stdout_lines}}
```

### Common return values
- e.g. "changed" = indicating if the task has to make changes

#### Parameterize your playbook
###Referencing Variables
- using double curly braces
- if you start a value with {{value}} , you must quote the whole expression to create valid YAML Syntax.

```bash
Before:

src: /usr/downloads/xyz.1.0.0.tgz

After:
if you are using initial with variable you need to enclose with double quotes
src: "{{node-app-location}}"
or
src: "{{location}}/xyz.1.0.0.tgz"
if you are using variable inside a path you don't need to enclose with double quotes

src: /usr/downloads/xyz.{{version}}.tgz

```
### Variables defined in a playbook 

Setting vars directly in playbook

```bash
1st method:
vars:
  - location: /usr/downloads
  - version: 1.0.1
  - node-app-location: /usr/downloads/xyz.1.0.0.tgz

```
### Passing variables On the command line
Setting vars at the command line

```bash
$: anisble-playbook -i hosts deploy-node.yml --extra-vars(it is euivalent to -e) "version: 1.0.1 location: /usr/downloads "
$: anisble-playbook -i hosts deploy-node.yml -e "version=1.0.1 location=/usr/downloads "
```

## External variables file
setting vars in an external variables file
- Variable file uses YAML Syntax
- so you can call it "project-vars.yaml"

```bash
project-vars file
version: 1.0.1
location: /usr/downloads

## We have to call this in each play
vars_files:
  - project-vars
```

## Git & Default inventory

Setup Git Project
- Create a git repository
- your playbooks should be stored safely in this remote repo
- Git repo is your "Single source of truth" for your ansible playbooks
- Great for working in teams

## Instead of using hosts in command use the below in ansible.cfg
```bash

$: mkdir ansible.cfg
[defaults]
host_key_checking = False
inventory = hosts

then use command : ansible-playbook deploy-node.yaml
```

## Ansible & Terraform
#### Integrate Ansible Playbook Execution in Terraform.




