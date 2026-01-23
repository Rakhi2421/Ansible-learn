## Project- Automate Node APP Deployment
Project Introduction : Deploy Node APP
- Steps we will go through
  1. Create an Instance
  2. Write Ansible Playbook
     - Install node & npm on server
     - copy node artifact and unpack
     - start application
     - Verify app running successfully.
```bash
deploy-node.yml
---
- name: Install node and npm
  hosts: 159.89.1.54
  tasks:
    - name: update apt repo and cache
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600
    - name: Install nodejs and npm
      apt:
        pkg:
          - nodejs
          - npm
- name: Deploy nodejs App
  hosts: 159.89.1.54
  tasks:
     - name: Copy nodejs folder to a server
       copy:
         src: /user/download/nodejs-app-1.0.0.tgz
         dest: /root/app-1.0.0.tgz
     - name: Unpack the nodejs file
       unarchive:
          src: /root/app-1.0.0.tgz
          dest: /root/ 
          remote_src: yes
             (or)
 - name: Deploy nodejs App
  hosts: 159.89.1.54
  tasks:
     - name: Unpack the nodejs file
       unarchive:
          src: /user/download/nodejs-app-1.0.0.tgz
          dest: /root/
     - name: Install dependencies
       npm:
         path: /root/package
     - name: Start the Application
       command:
         chdir: /root/package/app
         cmd: node server
         async: 1000
         poll: 0
     - name: Ensure App is running
       shell: ps aux | grep node
       register: app_status
     - debug: msg={{app_status}}

# create a new user and run the app using that new user

deploy-node.yml
---
- name: Install node and npm
  hosts: 159.89.1.54
  tasks:
    - name: update apt repo and cache
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600
    - name: Install nodejs and npm
      apt:
        pkg:
          - nodejs
          - npm

# create a new user
-name: Create a new linux user for node app
 hosts: 159.89.1.54
  tasks: 
    - name: Create Linux user
      user:
        name: rakhi
        comment: rakhi admin        = description for the user
        group: admin 
- name: Deploy nodejs App
  hosts: 159.89.1.54
  become: True
  become_user: rakhi
  tasks:
     - name: Unpack the nodejs file
       unarchive:
          src: /user/download/nodejs-app-1.0.0.tgz
          dest: /home/rakhi/
     - name: Install dependencies
       npm:
         path: /home/rakhi/package
     - name: Start the Application
       command:
         chdir: /home/rakhi/package/app
         cmd: node server
         async: 1000
         poll: 0
     - name: Ensure App is running
       shell: ps aux | grep node
       register: app_status
     - debug: msg={{app_status}}
```

## Project2- Automate NEXUS Deployment

Before Ansible, the manual steps we used:
- Create a server
- ssh into server and executed.
- Download Nexus binary and unpack
- Run Nexus application using Nexus user.
- Manual Installation and Manual Configuration.

This Project enables you to translate to ansible playbook.  
Learn common modules

```bash
---
- name: Install Java and net-tools
  hosts: 134.122.73.78
  tasks:
    - name: update apt repo and cache
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600
    - name: Install Java8
      apt: name=openjdk-8-jre-headless
    - name: Install net-tools
      apt: name=net-tools
- name: Download and unpack Nexus Installer 
  hosts: 134.122.73.78
  tasks:
    - name: Download Nexus
      get_url:
         url: https://download.sonatype.com/nexus/3/latest-unix.tar.gz
         dest: /opt/
         register: download_result
    - name: Untar Nexus Installer
      unarchive:
        src: {{donload_result.dest}}
        dest: /opt/
        remote_src: yes
    - name: Find Nexus Folder
      find:
        paths: /opt
        pattern: "nexus-*"
        file_type: directory
        register: find_result
    - name: Check if nexus folder already exists
      stat:
        path: /opt/nexus
      register: stat_result
    - name: Rename Nexus folder
      shell: mv{{find_result.files[0].path nexus}}
      when: not stat_result.stat.exists
- name: Create nexus to own nexus folders
  hosts: 134.122.73.78
  tasks:
    - name: Ensure group nexus exists
      group:
         name: nexus
         state: present
    - name: Create nexus user
      user:
         name: nexus
         group: nexus
    - name: Make nexus user owner of nexus folder
      file:
        path: /opt/nexus
        state: directory
        owner: nexus
        group: nexus
        recurse: yes
     - name: Make nexus user owner of sonatype-work folder
      file:
        path: /opt/sonatype-work
        state: directory
        owner: nexus
        group: nexus
        recurse: yes
- name: Start Nexus with nexus user
  hosts: 134.122.73.78
  become: True
  become_user: nexus
  tasks:
    - name: Set run as user nexus
      blockinfile:
        path: /opt/nexus/bin/nexus.rc
        block: |                                         ---> this task is to add a new lines in the file
          run_as_user="nexus"

tasks:
    - name: Set run as user nexus
      lineinfile:
        path: /opt/nexus/bin/nexus.rc
        regexp: '^#run_as_user=""'
        line: run_as_user="nexus"
   - name: Start nexus                                       ---> this task is to replace lines in the file
     command: /opt/nexus/bin/nexus start

- name: Verify nexus running
  hosts: nexus_user
  tasks:
    - name: Check with ps
      shell: ps aux| grep nexus
      register: app_status
    - debug: msg={{app_status.stdout_lines}}
    - name: Wait one minute
      pause:
        minutes: 1
    - name: Check with netstat
      shell: netstat -ntlp
      register: app_status
    - debug: msg={{app_status.stdout_lines}}     


    
