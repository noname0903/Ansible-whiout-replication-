What is MariaDB Replication?
MariaDB replication is the process of copying data from a master server to one or more slave servers. This allows for maintaining synchronized copies of a database across multiple servers, enabling various use cases.

Benefits of MariaDB Replication:

-Load Balancing: Slave servers can handle read queries, reducing the load on the master server.
-Scalability: Easily scale the system by adding more slave servers.
-Redundancy: Provides failover capabilities; if the master server goes down, a slave server can take over (with additional configuration).
-Data Backup: Data can be backed up from slave servers without impacting the master server's performance.
-Increased Performance: Separates write operations (on the master) from read operations (on the slaves).

Configure MariaDB Streaming Replication Between Two or More Nodes

This role was developed and tested for use with MariaDB to set up a redundant database backend. This configuration sets up a master/replica MariaDB setup with two nodes, enabling replication between them. It does not configure advanced clustering, but rather focuses on configuring a basic master-slave replication between two MariaDB nodes.

Add the replica database node to the Ansible inventory file and define for each database host
![image](https://github.com/user-attachments/assets/6cc004d5-86c2-4765-997d-a56aa1277d88)

![image](https://github.com/user-attachments/assets/8f7079a7-c7be-47cb-9245-c4fade7af45e)

