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

