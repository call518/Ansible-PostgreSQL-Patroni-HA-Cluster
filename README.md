    Architecture Overview

    The cluster consists of:
    1 Ansible Manager Node
    1 Haproxy Node
    1 etcd Node
    2 PostgreSQL Nodes managed by Patroni



                        +-------------------+
                        | ansible-manager   |  <-- Runs Ansible playbooks
                        +-------------------+

                                |
                                v
                        +-------------------+
                        |     haproxy       |  <-- Load balancer (RW:5000, RO:5001)
                        +-------------------+
                                |
          +---------------------+----------------------+
          |                                            |
     +---------+                              +---------+
     |  node1  |  <-- PostgreSQL + Patroni    |  node2  |  <-- PostgreSQL + Patroni
     +---------+                              +---------+

                        +---------+
                        |  etcd1  |  <-- Key-value store for Patroni
                        +---------+


    How to Use 
    -----------

    git clone https://github.com/ozk17/ansible-patroni-cluster.git
    # Passwordless ssh each node

    ansible-playbook -i inventory.ini setup_etcd.yml # setup etcd

    etcdctl --endpoints=172.26.228.41:2379 endpoint status --write-out=table # chech etcd status
    +--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
    |      ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
    +--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
    | 172.26.228.41:2379 | 9f4ab1e673198e09 |  3.5.14 |   20 kB |      true |      false |         2 |          4 |                  4 |        |
    +--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+


    ansible-playbook -i inventory.ini setup_postgres_patroni.yml # setup patroni 

     /opt/patroni-venv/bin/patronictl -c /etc/patroni.yml list # chech patroni list
 
    +  Cluster: postgres (7525717775729802398) +-----------+----+-----------+
    | Member        | Host          | Role    | State     | TL | Lag in MB |
    +---------------+---------------+---------+-----------+----+-----------+
    | 172.26.226.99 | 172.26.226.99 | Leader  | running   |  1 |           |
    | 172.26.235.51 | 172.26.235.51 | Replica | streaming |  1 |         0 |
    +---------------+---------------+---------+-----------+----+-----------+

    ansible-playbook -i inventory.ini haproxy-setup.yml
