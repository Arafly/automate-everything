# Ansible Configuration Management - Automate Project 7 to 10

In Projects 7 to 10 we've had to perform a lot of manual operations - setting up virtual servers, installation and configuration of required software, deployment of  web application.

We're about to automate the routine tasks with a Configuration Management Tool called *Ansible*

### Task
1. Install and configure Ansible client to act as a Jump Server/Bastion Host
2. Create a simple Ansible playbook to automate servers configuration

## Ansible Client as a Jump Server (Bastion Host)

A Jump Server (also referred to as Bastion Host) is an intermediary server through which access to internal network can be provided. Ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even authorized persons cannot SSH into the Web servers directly and can only access it through a Bastion Host - it provide better security and reduces attack surface.

In the diagram below the Virtual Private Network (VPC) is divided into two subnets - Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

*image archi