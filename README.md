    Architecture Overview

    The cluster consists of:
    1 Ansible Manager Node
    1 Haproxy Node
    1 etcd Node
    2 PostgreSQL Nodes managed by Patroni


    Hostnames:
    192.168.156.101    pgsql-ha-haproxy
    192.168.156.102    pgsql-ha-etcd
    192.168.156.103    pgsql-ha-01
    192.168.156.104    pgsql-ha-02


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

    etcdctl --endpoints=192.168.156.102:2379 endpoint status --write-out=table # chech etcd status
    +----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
    |       ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
    +----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
    | 192.168.156.102:2379 | 4f55cac6744452f2 |  3.5.14 |   25 kB |      true |      false |         6 |         12 |                 12 |        |
    +----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+



    ansible-playbook -i inventory.ini setup_postgres_patroni.yml # setup patroni 

    /opt/patroni-venv/bin/patronictl -c /etc/patroni.yml list # chech patroni list
    + Cluster: postgres (7547525253919397715) ----+-----------+----+-----------+
    | Member          | Host            | Role    | State     | TL | Lag in MB |
    +-----------------+-----------------+---------+-----------+----+-----------+
    | 192.168.156.103 | 192.168.156.103 | Leader  | running   |  1 |           |
    | 192.168.156.104 | 192.168.156.104 | Replica | streaming |  1 |         0 |
    +-----------------+-----------------+---------+-----------+----+-----------+

    ansible-playbook -i inventory.ini haproxy-setup.yml
