# Ansible Configuration Management - Automate Project 7 to 10

In Projects 7 to 10 we've had to perform a lot of manual operations - setting up virtual servers, installation and configuration of required software, deployment of  web application.

We're about to automate the routine tasks with a Configuration Management Tool called *Ansible*

### Task
1. Install and configure Ansible client to act as a Jump Server/Bastion Host
2. Create a simple Ansible playbook to automate servers configuration

## Ansible Client as a Jump Server (Bastion Host)

A Jump Server (also referred to as Bastion Host) is an intermediary server through which access to internal network can be provided. Ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even authorized persons cannot SSH into the Web servers directly and can only access it through a Bastion Host - it provide better security and reduces attack surface.

In the diagram below the Virtual Private Network (VPC) is divided into two subnets - Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

![](https://github.com/Arafly/automate-everything/blob/master/assets/bastion.png)

### Install and configure Ansible on VM
- You can update the name tag on your Jenkins instance to *Jenkins-Ansible* if you like. As we'll also use this server to run ansible playbooks.
- Create a new repo on your GitHub and name it whatever is suitable (mine is automate-everything).
- Install Ansible
  
```
sudo apt update

sudo apt install ansible
```

Check your Ansible version by running 
`ansible --version`

```
Output:

ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/araflyayinde/.ansible/plugins/modules', '
/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
```

### Configure Jenkins build job to save your repository content every time you change it. 

- Create a new Freestyle project "ansible" in Jenkins and point it to your repository.

> You can first place in your project name and clone the *tooling* project. Like this:

![](https://github.com/Arafly/automate-everything/blob/master/assets/jenkins_clone.PNG)

![](https://github.com/Arafly/automate-everything/blob/master/assets/third_build.PNG)

- Configure Webhook in GitHub and set webhook to trigger ansible build.
- Configure a Post-build job to save all (**) files.

*image third_build

- Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder:
  
`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

```
$ ls /var/lib/jenkins/jobs/ansible/builds/
1  2  3  legacyIds  permalinks

$ ls /var/lib/jenkins/jobs/ansible/builds/3
archive  build.xml  changelog.xml  log  polling.log

$ ls /var/lib/jenkins/jobs/ansible/builds/3/archive/
README.md  assets  config_mgt.md
```

Now our setup will look like this:

![](https://github.com/Arafly/automate-everything/blob/master/assets/jenkins_ansible.png)

## Ansible Development
- In your  GitHub repository, create a new branch that will be used for development of a new feature (mine is ansible_integration)

- Checkout the newly created feature branch to your local machine to start building your code and directory structure
- Create a directory and name it *playbooks* - it will be used to store all your playbook files. Also Within the playbooks folder, create your first playbook, and name it *common.yml*
- Create a directory and name it *inventory* - it will be used to keep your hosts organised. Within the inventory folder, create an inventory file (use .ini format) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

> Establish an ssh connection to each ip addresses grouped in the inventory file

### Set up an Ansible Inventory
An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. It is important to have a way to organize our hosts in such an Inventory.

Below inventory structure in the inventory/dev file to start configuring your development servers. *Ensure to replace the IP addresses according to your own setup*.

Update your inventory/dev.yml file with this snippet of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

### Create a Playbook
It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.

Update your playbooks/common.yml file with following code:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    yum:
      name: wireshark
      state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    apt:
      name: wireshark
      state: latest
```

This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your Centos8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.


### Update GIT with the latest code

Now that we have a separate branch, we'll need to raise a Pull Request (PR), so as to get your branch peer merged to the master branch.

use git commands to add, commit and push your branch to GitHub.

```
git status

git add <selected files>

git commit -m "commit message"
```
- Create a Pull request (PR)
Wear a hat of another developer for a second, and act as a reviewer. If the reviewer is happy with your new feature development, merge the code to the master branch.

![](https://github.com/Arafly/automate-everything/blob/master/assets/PR_new.PNG)

![](https://github.com/Arafly/automate-everything/blob/master/assets/pr.PNG)

![](https://github.com/Arafly/automate-everything/blob/master/assets/merge_pr.PNG)


Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes using `git pull`.
Once your code changes appear in master branch - Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.

Update your inventory file with the list of remote host you're targeting i.e lb, webservers, db

- Setting Up Host Aliases
We'd use aliases to name servers in a way that facilitates referencing those servers later, when running commands and playbooks. And also to make it more friendly instead of seeing bunch of ugly ip addresses.

To use an alias, include a variable named *ansible_host* after the alias name, containing the corresponding IP address or hostname of the server that should respond to that alias.

```
[nfs]
file-storage ansible_host=34.105.245.167 ansible_ssh_user='araflyayinde'

[webservers]
webserver1 ansible_host=34.105.163.196 ansible_ssh_user='araflyayinde'
webserver2 ansible_host=35.242.128.108 ansible_ssh_user='araflyayinde'

[db]
mysql ansible_host=34.145.122.179 ansible_ssh_user='araflyayinde'

[lb]
nginx ansible_host=34.105.133.81 ansible_ssh_user='araflyayinde'
```

Now run the follwing command to list out your inventory:

`$ ansible-inventory -i /var/lib/jenkins/jobs/ansible/builds/9/archive/inventory/dev --list`

``

```
Output

{
    "_meta": {
        "hostvars": {
            "file-storage": {
                "ansible_host": "34.105.245.167",
                "ansible_ssh_user": "araflyayinde"
            },
            "mysql": {
                "ansible_host": "34.145.122.179",
                "ansible_ssh_user": "araflyayinde"
            },
            "nginx": {
                "ansible_host": "34.105.133.81",
                "ansible_ssh_user": "araflyayinde"
            },
            "webserver1": {
                "ansible_host": "34.105.163.196",
                "ansible_ssh_user": "araflyayinde"
            },
            "webserver2": {
                "ansible_host": "35.242.128.108",
                "ansible_ssh_user": "araflyayinde"
            }
        }
    },
    "all": {
        "children": [
            "db",
            "lb",
            "nfs",
            "ungrouped",
            "webservers"
        ]
    },
   "db": {
        "hosts": [
            "mysql"
        ]
    },
    "lb": {
        "hosts": [
            "nginx"
        ]
    },
    "nfs": {
        "hosts": [
            "file-storage"
        ]
    },
    "webservers": {
        "hosts": [
            "webserver1",
            "webserver2"
        ]
    }
}
```

- Run a ping to test connection to all our remote hosts

`$ ansible all -i /var/lib/jenkins/jobs/ansible/builds/9/archive/inventory/dev -m ping`
 
This would execute the ping module on all hosts listed in your custom inventory file.

```
Output

mysql | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
file-storage | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
webserver1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
webserver2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

> If you observe, in the inventory file, there's ansible_ssh_host repeated on all five servers. DRY Principle (Don't Repeat Yourself) would frown at this, exactly like this:

![](https://github.com/Arafly/automate-everything/blob/master/assets/photo_2021-05-22_18-06-46.jpg)

So, what we have to do is further regroup our hosts and append the single ssh_user to them: 

```
[pr-11servers]
file-storage ansible_host=34.105.245.167
webserver1 ansible_host=34.105.163.196
webserver2 ansible_host=35.242.128.108
mysql ansible_host=34.145.122.179 
nginx ansible_host=34.105.133.81 

[pr-11servers:vars]
ansible_ssh_user='araflyayinde'

```

- Update your playbooks/common.yml file with following code:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    yum:
      name: wireshark
      state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    apt:
      name: wireshark
      state: latest
```

## Run first Ansible test
Now, it's time to execute ansible-playbook command and verify if your playbook actually works:

`ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml`

When I ran the above command, I got the error message below. 

```
$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/9/archive/inventory/dev /var/lib/jenkins/jobs/ansible/builds/9/archive/playbooks/common.yml
PLAY [update web, nfs and db servers] *****************************************************************************************************************************
TASK [Gathering Facts] ********************************************************************************************************************************************
ok: [webserver1]
ok: [webserver2]
ok: [file-storage]
ok: [mysql]
TASK [ensure wireshark is at the latest version] ******************************************************************************************************************
ok: [mysql]
ok: [file-storage]
ok: [webserver1]
ok: [webserver2]
PLAY [update Nginx LB server] *************************************************************************************************************************************
TASK [Gathering Facts] ********************************************************************************************************************************************
ok: [nginx]
TASK [ensure wireshark is at the latest version] ******************************************************************************************************************
ok: [nginx]
PLAY RECAP ********************************************************************************************************************************************************
file-storage               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
mysql                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
nginx                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webserver1                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webserver2                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

You can go to each of the servers and check if wireshark has been installed by running `which wireshark` or `wireshark --version`

Your updated with Ansible architecture now looks like this:

![](https://github.com/Arafly/automate-everything/blob/master/assets/ansible_architecture.png)

### **Congratulations!**

