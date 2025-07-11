Architecture Overview
The cluster consists of:
1 Ansible Manager Node
1 etcd Node
2 PostgreSQL Nodes managed by Patroni

+-------------------+
| ansible-manager   | <-- Runs Ansible playbooks
+-------------------+



+---------+    +---------+
| etcd1   |    | node1   | <-- PostgreSQL + Patroni
+---------+    +---------+
                   |


            
               +---------+
               | node2   | <-- PostgreSQL + Patroni
               +---------+


Step-by-Step Setup
Create Virtual Machines with Multipass
multipass launch --name ansible-manager
multipass launch --name etcd1
multipass launch --name node1
multipass launch --name node2
Check their IPs:


multipass list
Name                    State             IPv4             Image
ansible-manager         Running           172.26.238.104   Ubuntu 24.04 LTS
etcd1                   Running           172.26.228.41    Ubuntu 24.04 LTS
node1                   Running           172.26.226.99    Ubuntu 24.04 LTS
node2                   Running           172.26.235.51    Ubuntu 24.04 LTS
Prepare the Ansible Manager
Shell into the manager node: (Apply this per node)
multipass shell ansible-manager 
Create the ansible user and set a password:
sudo useradd -m -s /bin/bash ansible
sudo passwd ansible
Enable SSH password authentication:
sudo vi /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
# Set: PasswordAuthentication yes

sudo systemctl restart ssh
Allow passwordless sudo for Ansible:
sudo vi /etc/sudoers.d/90-cloud-init-users
# Add: ansible ALL=(ALL) NOPASSWD:ALL
Setup SSH Keys
Login as the ansible user:
su - ansible
Generate and copy SSH keys to all nodes:
ssh-keygen -t rsa -b 4096

ssh-copy-id ansible@172.26.228.41
# ssh-copy-id ansible@<etcd1-ip>

ssh-copy-id ansible@172.26.226.99
#ssh-copy-id ansible@<node1-ip>

ssh-copy-id ansible@172.26.235.51
ssh-copy-id ansible@<node2-ip>
Clone the Ansible Playbook Repository
First, clone the repository that contains all the Ansible automation needed for this cluster setup:
git clone https://github.com/ozk17/ansible-patroni-cluster.git
cd ansible-patroni-cluster
This repository is designed to be clean and modular, making it easy to understand, modify, and reuse. It includes everything needed to bootstrap both the etcd and Patroni-based PostgreSQL cluster using Ansible.
Repository Structure
Here are the main files and templates included in the repository:
ansible-patroni-cluster/
├── inventory.ini                 # Inventory file to define hosts
├── setup_etcd.yml                # Playbook to install and configure etcd
├── setup_postgres_patroni.yml    # Playbook to install PostgreSQL and Patroni
└── templates/                    # Jinja2 templates for configuration
    ├── etcd.conf.j2              # etcd configuration template
    ├── patroni.service.j2        # systemd service file for Patroni
    ├── patroni.yml.j2            # Patroni configuration template

Each file has a specific role and is designed to be easily customizable for your own cluster needs. I'll walk you through the content and purpose of each of these files in the next sections.
Install Ansible
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
Run the Playbooks
Edit your inventory.ini:
[etcd]
172.26.228.41

[postgresql]
172.26.226.99
172.26.235.51
Run etcd setup:
ansible-playbook -i inventory.ini setup_etcd.yml
Check etcd status:
root@etcd1:~# etcdctl --endpoints=172.26.228.41:2379 endpoint status --write-out=table
+--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|      ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 172.26.228.41:2379 | 9f4ab1e673198e09 |  3.5.14 |   20 kB |      true |      false |         2 |          4 |                  4 |        |
+--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
root@etcd1:~#
Run Patroni + PostgreSQL setup:
ansible-playbook -i inventory.ini setup_postgres_patroni.yml
Verify the Cluster
SSH into node1 or node2 and run:
/opt/patroni-venv/bin/patronictl -c /etc/patroni.yml list
Sample output:
+ Cluster: postgres (7525717775729802398) +-----------+----+-----------+
| Member        | Host          | Role    | State     | TL | Lag in MB |
+---------------+---------------+---------+-----------+----+-----------+
| 172.26.226.99 | 172.26.226.99 | Leader  | running   |  1 |           |
| 172.26.235.51 | 172.26.235.51 | Replica | streaming |  1 |         0 |
+---------------+---------------+---------+-----------+----+-----------+
Let me know your thoughts or issues in the GitHub repo. If you liked this article, feel free to follow me for more content on PostgreSQL, HA architectures, and DevOps best practices.
