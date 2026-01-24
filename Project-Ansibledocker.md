# Project - Ansible & Docker

Steps we will go throgh

1. Create an AWS EC2 Instance.
2. Write Ansible Playbook.
 - Install docker & docker-compose
 - Start docker containers to run applications

Overview:
- Create an AWS Instance with terraform.
- Configure Inventory file to connect to AWS EC2 Instance.
- Install docker and docker-compose
- Copy docker - compose file to server.
- Start docker containers

```bash
---
- name: Install python3, docker, docker-compose
  hosts: docker_server
  become: yes
  gather_facts: False  
  tasks:
    - name: Install python3 and docker
      vars:
        ansible_python_interpreter: /usr/bin/python
      yum:
        name:
          - python3
          - docker
        update_cache: yes
        state: present

    - name: Install Docker-compose
      get_url:
        url: https://github.com/docker/compose/releases/download/1.27.4/docker-compose-Linux-{{lookup('pipe','uname -m')}}
        dest: /usr/local/bin/docker-compose
        mode: +x
    - name: Start docker daemon
      systemd:
        name: docker
        state: started
    - name: Install docker python module
      pip:
        name:
          - docker
          - docker-compose

- name: Add ec2-user to docker group
  hosts: docker_server
  become: yes
  tasks:
    - name: Add ec2-user to docker group
      user:
        name: ec2-user
        groups: docker
        append: yes
    - name: Reconnect to server session
      meta: reset connection

- name: Create a new Linux user
  hosts: docker_server
  become: yes
  tasks:
    - name: Create new Linux user
      user:
        name: nana
        groups: adm,docker

- name: Start docker containers
  hosts: docker_server
  vars_prompt:
    - name: docker_password
      prompt: Enter password for docker registry
  become: yes
  tasks:
    - name: Copy docker compose
      copy:
        src: /users/rakhi/downloads/docker-compose.yaml
        dest: /home/ec2-user/docker-compose.yaml
     - name: Docker Login
       docker_login:
         registry_url: https://index.docker.io/v1/
         username: rakesh
         password: "{{docker_password}}"

