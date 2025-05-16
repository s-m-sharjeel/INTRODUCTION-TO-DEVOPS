# MySQL Database Automation with Ansible

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A DevOps project that automates the installation and configuration of MySQL database servers using Ansible. This project demonstrates infrastructure as code (IaC) principles by automating the setup of MySQL database servers, creating databases and users with appropriate privileges.

## Project Overview

This project automates the following tasks:
- Installation of MySQL server on target hosts
- Configuration of a root password
- Creation of a database
- Creation of a database user with appropriate privileges

## Prerequisites

- AWS EC2 instances (controller and database servers)
- Ubuntu OS on all instances
- SSH access to all instances
- `devops.pem` private key for SSH authentication

## Environment Setup

### 1. Setting up EC2 Instances

Create two EC2 instances in AWS:
- Controller node (Ansible host)
- Database server node

Ensure both instances:
- Use Ubuntu as the operating system
- Are in the same subnet
- Can be accessed using the `devops.pem` key

### 2. Instance Configuration

SSH into each instance and configure hostnames:

```bash
# For controller instance
ssh -i devops.pem ubuntu@<controller_ip_address>
sudo hostname controller
exit

# For database server instance
ssh -i devops.pem ubuntu@<db_ip_address>
sudo hostname db1
exit
```

Re-SSH into the instances to apply the hostname changes.

### 3. Ansible Installation (Controller Node)

Install Ansible on the controller node:

```bash
ssh -i devops.pem ubuntu@<controller_ip_address>
sudo su -
apt update
sudo apt-add-repository -y ppa:ansible/ansible
sudo apt-get install -y ansible
sudo apt update
ansible --version
```

### 4. Ansible Configuration

Configure the Ansible hosts file to include the database servers:

```bash
cd /etc/ansible/
nano hosts
```

Add the following to the hosts file:

```
[dbservers]
db ansible_host=<db_ip_address>
```

### 5. SSH Key Setup for Passwordless Authentication

Generate SSH keys and copy to target hosts:

```bash
cd ~/.ssh/
ssh-keygen

# Copy the public key to the database server
cat ~/.ssh/id_rsa.pub | ssh -i ~/project/devops.pem ubuntu@<db_ip_address> "cat >> ~/.ssh/authorized_keys"
```

### 6. Project Setup

Create a project directory and copy the required SSH key:

```bash
mkdir ~/project
cd ~/project
nano devops.pem
# Paste the contents of your devops.pem key here
chmod 600 ~/project/devops.pem
```

## Playbook Structure

The main playbook `mysql_setup.yaml` contains tasks to:
1. Install required Python MySQL libraries
2. Preconfigure MySQL root password
3. Install MySQL server
4. Ensure MySQL service is running
5. Create a database
6. Create a MySQL user with appropriate privileges

```yaml
---
- name: Automate MySQL installation and configuration
  hosts: db
  become: yes
  vars:
    mysql_root_password: "YourRootPassword"
  tasks:
    - name: Install required Python MySQL libraries
      package:
        name: "{{ item }}"
        state: present
      loop:
        - python3-pymysql

    - name: Preconfigure MySQL password
      debconf:
        name: "mysql-server"
        question: "mysql-server/root_password"
        value: "{{ mysql_root_password }}"
        vtype: "password"

    - name: Preconfigure MySQL password confirmation
      debconf:
        name: "mysql-server"
        question: "mysql-server/root_password_again"
        value: "{{ mysql_root_password }}"
        vtype: "password"

    - name: Install MySQL server
      apt:
        name: mysql-server
        state: present
        update_cache: yes

    - name: Ensure MySQL service is running
      service:
        name: mysql
        state: started
        enabled: true

    - name: Create a new database
      mysql_db:
        name: my_database
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: Create a new MySQL user
      mysql_user:
        name: my_user
        password: 'MySecurePassword'
        priv: 'my_database.*:ALL'
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
```

## Running the Playbook

Execute the playbook from the controller node:

```bash
cd ~/project
ansible-playbook mysql_setup.yaml
```

## Verification

Verify the MySQL installation and configuration on the database server:

```bash
# Check MySQL service status
systemctl status mysql

# Log in as root
mysql -u root -p
# Enter password: YourRootPassword

# Log in as the created user and access the database
mysql -u my_user -p -D my_database
# Enter password: MySecurePassword
```

## Security Considerations

- Store sensitive information like passwords securely using Ansible Vault
- Consider implementing more restrictive user privileges based on requirements
- Implement proper network security groups for database access

## Future Improvements

- Add database backups automation
- Implement high availability configurations
- Add monitoring and alerting setup
- Create roles for better code organization
- Implement parameterized configurations for different environments

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Author

Shaikh M. Sharjeel
