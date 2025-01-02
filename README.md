# INTRODUCTION-TO-DEVOPS
Automating the installation of a MySQL database server and the creation of a user and database using Ansible.

create 2 ec2 instances on aws
- use devops.pem key
- use the same subnet

ssh into the machine using:
ssh -i devops.pem ubuntu@<ip_address>

change hostname to controller and db1 and db2:
sudo hostname <hostname>
re-ssh into the vms to get the changes

on the db machine show that MySQL hasn't been installed yet:
systemctl status mysql

sudo su -
apt update

sudo apt-add-repository -y ppa:ansible/ansible
sudo apt-get install -y ansible

sudo apt update
sudo apt install ansible

ansible --version

cd /etc/ansible/ 
hosts has all the targeted hosts

nano hosts
[dbservers]
db ansible_host=<insert ip for db server wali vm>

try pinging...
ansible dbservers -m ping -u ubuntu
ping should fail

exit root
cd .ssh/
ssh-keygen

create project dir
mkdir project
cd project

nano devops.pem
copy devops key here to ssh into the host machines
change the privilege of the devops file
chmod 600 ~/project/devops.pem
ls -l ~/project/devops.pem
will show privilege^

cat ~/.ssh/id_rsa.pub | ssh -i ~/project/devops.pem ubuntu@<vm_ip> "cat >> ~/.ssh/authorized_keys"

nano mysql_setup.yaml
---
- name: Automate MySQL installation and configuration
  hosts: db
  become: yes
  vars:
    mysql_root_password: "YourRootPassword"

  tasks:
    # Install required Python MySQL libraries
    - name: Install required Python MySQL libraries
      package:
        name: "{{ item }}"
        state: present
      loop:
        - python3-pymysql

    # Preconfigure MySQL password
    - name: Preconfigure MySQL password
      debconf:
        name: "mysql-server"
        question: "mysql-server/root_password"
        value: "{{ mysql_root_password }}"
        vtype: "password"

    # Preconfigure MySQL password confirmation
    - name: Preconfigure MySQL password confirmation
      debconf:
        name: "mysql-server"
        question: "mysql-server/root_password_again"
        value: "{{ mysql_root_password }}"
        vtype: "password"

    # Install MySQL server
    - name: Install MySQL server
      apt:
        name: mysql-server
        state: present
        update_cache: yes

    # Ensure MySQL service is running
    - name: Ensure MySQL service is running
      service:
        name: mysql
        state: started
        enabled: true

    # Create a database
    - name: Create a new database
      mysql_db:
        name: my_database
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    # Create a MySQL user
    - name: Create a new MySQL user
      mysql_user:
        name: my_user
        password: 'MySecurePassword'
        priv: 'my_database.*:ALL'
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

no need for inventory.yaml cus the hosts are already stored in the etc/ansible/hosts file (default)

executing playbook:
ansible-playbook mysql_setup.yaml

testing on the db machine:
systemctl status mysql
mysql -u root -p
password: YourRootPassword
mysql -u my_user -p -D my_database
password: MySecurePassword
