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

- Configure Jenkins build job to save your repository content every time you change it. 
Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
Configure Webhook in GitHub and set webhook to trigger ansible build.
Configure a Post-build job to save all (**) files, like you did it in Project 9.
Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
Note: Trigger Jenkins project execution only for /main (master) branch.

Now your setup will look like this: