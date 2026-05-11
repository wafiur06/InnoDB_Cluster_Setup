# MySQL 8.4 LTS InnoDB Cluster with High Availability (WafiCluster)

This repository provides a step-by-step guide and configuration for deploying a **3-node MySQL InnoDB Cluster** using **Group Replication** and **MySQL Router** on Oracle Linux 9.

---

## 🏗️ Architecture Overview

The cluster is designed for **High Availability (HA)**. It ensures that if the Primary node fails, a Secondary node is automatically promoted to Primary without manual intervention.

- **Primary Node (R/W):** Handles all write operations and data consistency.
- **Secondary Nodes (R/O):** Act as hot-standbys and handle read-only traffic.
- **MySQL Router:** Serves as the intelligent gateway, routing traffic to the correct nodes.

### 🖥️ Node Details
| Node Name | IP Address     | Role      | OS             |
| --------- | -------------- | --------- | -------------- |
| dc-node1  | 192.168.56.11  | Primary   | Oracle Linux 9 |
| dc-node2  | 192.168.56.12  | Secondary | Oracle Linux 9 |
| dc-node3  | 192.168.56.13  | Secondary | Oracle Linux 9 |

---

## ⚙️ Prerequisites

* **Environment:** Vagrant & VirtualBox
* **Operating System:** Oracle Linux 9
* **User Privileges:** Sudo access for `vagrant` user
* **Networking:** Host-only adapter (192.168.56.0/24 range)

---

## 🌐 1. Network & Firewall Setup

Run these commands on **all nodes** to ensure seamless communication:

```bash
# Set up hostnames
sudo vi /etc/hosts
# Add the following:
192.168.56.11 dc-node1
192.168.56.12 dc-node2
192.168.56.13 dc-node3

# Open cluster-specific ports
sudo firewall-cmd --permanent --add-port={3306,33060,33061,33062}/tcp
sudo firewall-cmd --reload

2. Installation (All Nodes)

Install the official MySQL 8.4 LTS repository and the core components:

# Add MySQL Repository
sudo dnf install -y [https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm](https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm)

# Install MySQL Server, Shell, and Router
sudo dnf install -y mysql-community-server mysql-shell mysql-router-community

# Enable and start MySQL
sudo systemctl enable --now mysqld

⚡ 3. Cluster Initialization
A. Initialize the Seed Node (dc-node1)

Access the MySQL Shell to configure the first instance:
mysqlsh --uri root@192.168.56.11:3306
// Configure instance for cluster compatibility
dba.configureInstance('root@192.168.56.11:3306')

// Create the cluster named 'WafiCluster'
var cluster = dba.createCluster('WafiCluster');

B. Add Secondary Nodes

Expand the cluster by adding dc-node2 and dc-node3. When prompted for recovery, choose [C]lone.
cluster.addInstance('root@192.168.56.12:3306')
cluster.addInstance('root@192.168.56.13:3306')

To copy the README content for GitHub, you should use the raw Markdown format to ensure all the formatting, tables, and code blocks stay intact.
How to Copy:

    Select the text inside the box below.

    Copy (Ctrl+C or Cmd+C).

    Go to your GitHub repository, click Add file > Create new file.

    Name it README.md.

    Paste (Ctrl+V or Cmd+V) and commit the changes.

README.md
Markdown

# MySQL 8.4 LTS InnoDB Cluster with High Availability (WafiCluster)

This repository provides a step-by-step guide and configuration for deploying a **3-node MySQL InnoDB Cluster** using **Group Replication** and **MySQL Router** on Oracle Linux 9.

---

## 🏗️ Architecture Overview

The cluster is designed for **High Availability (HA)**. It ensures that if the Primary node fails, a Secondary node is automatically promoted to Primary without manual intervention.

- **Primary Node (R/W):** Handles all write operations and data consistency.
- **Secondary Nodes (R/O):** Act as hot-standbys and handle read-only traffic.
- **MySQL Router:** Serves as the intelligent gateway, routing traffic to the correct nodes.

### 🖥️ Node Details
| Node Name | IP Address     | Role      | OS             |
| --------- | -------------- | --------- | -------------- |
| dc-node1  | 192.168.56.11  | Primary   | Oracle Linux 9 |
| dc-node2  | 192.168.56.12  | Secondary | Oracle Linux 9 |
| dc-node3  | 192.168.56.13  | Secondary | Oracle Linux 9 |

---

## ⚙️ Prerequisites

* **Environment:** Vagrant & VirtualBox
* **Operating System:** Oracle Linux 9
* **User Privileges:** Sudo access for `vagrant` user
* **Networking:** Host-only adapter (192.168.56.0/24 range)

---

## 🌐 1. Network & Firewall Setup

Run these commands on **all nodes** to ensure seamless communication:

```bash
# Set up hostnames
sudo vi /etc/hosts
# Add the following:
192.168.56.11 dc-node1
192.168.56.12 dc-node2
192.168.56.13 dc-node3

# Open cluster-specific ports
sudo firewall-cmd --permanent --add-port={3306,33060,33061,33062}/tcp
sudo firewall-cmd --reload

📦 2. Installation (All Nodes)

Install the official MySQL 8.4 LTS repository and the core components:
Bash

# Add MySQL Repository
sudo dnf install -y [https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm](https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm)

# Install MySQL Server, Shell, and Router
sudo dnf install -y mysql-community-server mysql-shell mysql-router-community

# Enable and start MySQL
sudo systemctl enable --now mysqld

⚡ 3. Cluster Initialization
A. Initialize the Seed Node (dc-node1)

Access the MySQL Shell to configure the first instance:
Bash

mysqlsh --uri root@192.168.56.11:3306

Inside MySQL Shell (JS > mode):
JavaScript

// Configure instance for cluster compatibility
dba.configureInstance('root@192.168.56.11:3306')

// Create the cluster named 'WafiCluster'
var cluster = dba.createCluster('WafiCluster');

B. Add Secondary Nodes

Expand the cluster by adding dc-node2 and dc-node3. When prompted for recovery, choose [C]lone.
JavaScript

cluster.addInstance('root@192.168.56.12:3306')
cluster.addInstance('root@192.168.56.13:3306')

🚦 4. MySQL Router Setup

Bootstrap the MySQL Router on dc-node1 to act as the traffic load balancer.
# Bootstrap the router against the cluster
sudo mysqlrouter --bootstrap root@192.168.56.11:3306 --user=mysqlrouter

# Enable and start the service
sudo systemctl enable --now mysqlrouter


📊 5. Monitoring and Verification

To verify the cluster health, run the following in MySQL Shell:

var cluster = dba.getCluster('WafiCluster')
cluster.status()

