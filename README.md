Architecture Overview


The cluster consists of:
1 Ansible Manager Node
1 etcd Node
2 PostgreSQL Nodes managed by Patroni

+-------------------+
| ansible-manager   | <-- Runs Ansible playbooks
+-------------------+
                |
                
+---------+    +---------+
| etcd1   |    | node1   | <-- PostgreSQL + Patroni
+---------+    +---------+
                   |
                   
               +---------+
               | node2   | <-- PostgreSQL + Patroni
               +---------+




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
