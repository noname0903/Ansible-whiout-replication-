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
* Notice:
  Change ansible_host="your IP"
Here's request configure
![image](https://github.com/user-attachments/assets/2a613bcb-ba12-433f-b655-20cac2623455)

This is the complete configuration when moved back into 1 playbook
![image](https://github.com/user-attachments/assets/ee42228d-82d9-4302-9c8d-af83af829cb0)
![image](https://github.com/user-attachments/assets/e02a830e-6f8c-44be-8b3d-2ad413a485b3)
![image](https://github.com/user-attachments/assets/76118446-db91-4282-962b-63d14a2882a8)
![image](https://github.com/user-attachments/assets/d6764da9-b87e-4421-8c5c-378b063b3a57)
![image](https://github.com/user-attachments/assets/c46f09e8-da4b-4ff7-8ae9-29c603521487)

*Result
![image](https://github.com/user-attachments/assets/ce3deea2-d2a3-4669-8213-c9f658256cce)

+)Master
![image](https://github.com/user-attachments/assets/de09ddf6-4d9b-4814-93d3-90bf5c73bfb9)
![image](https://github.com/user-attachments/assets/9f3a8751-e3d9-447c-8c43-4aee0623731d)
+)Slave
![image](https://github.com/user-attachments/assets/de09ddf6-4d9b-4814-93d3-90bf5c73bfb9)
![image](https://github.com/user-attachments/assets/7b630a75-e7f6-4519-8f80-b62751ecf950)



       

