# 🗄️ WafiCluster: MySQL 8.4 LTS InnoDB Cluster Setup

This repository contains the complete documentation and configuration for deploying a 3-node MySQL InnoDB Cluster with High Availability (HA) and Automatic Failover on Oracle Linux 9.

---

## Cluster Overview

This setup consists of **three MySQL nodes** configured in a **high-availability cluster** using **MySQL Shell (mysqlsh)**.

### 🏗️ Architecture Diagram

```
                  ┌────────────────────┐
                  │   MySQL Shell      │
                  │   (Cluster Admin)  │
                  │   dc-node1         │
                  └─────────┬──────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  dc-node1    │   │  dc-node2    │   │  dc-node3    │
│  192.168.56.11 │ │ 192.168.56.12 │ │ 192.168.56.13 │
│  PRIMARY      │   │  SECONDARY   │   │  SECONDARY   │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                  │
       └────────── Group Replication ────────┘
                     (Port 33061)
```

---

## Node Details
```
| Node Name | IP Address    | Role      |
| --------- | ------------- | --------- |
| dc-node1  | 192.168.56.11 | Primary   |
| dc-node2  | 192.168.56.12 | Secondary |
| dc-node3  | 192.168.56.13 | Secondary |
```
---

## ⚙️ Prerequisites

* OS: Oracle Linux 9
* User: Sudo privileges for the vagrant user
* Networking: Connectivity via a Host-only adapter (192.168.56.0/24)
* SELinux enabled (configured properly)
* Firewall: Allowed traffic on ports 3306, 33060, 33061, and 33062

---

## 🌐 1. Network Configuration

Edit `sudo vi /etc/hosts` on **all nodes** to enable hostname resolution:

```bash
192.168.56.11 dc-node1
192.168.56.12 dc-node2
192.168.56.13 dc-node3
```

---

## 2. Install the MySQL Community Repository in all node

```bash
# Install MySQL Repository
sudo dnf install -y https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm

# Install MySQL Server, Shell, and Router
sudo dnf install -y mysql-community-server mysql-shell mysql-router-community

# Start and Enable MySQL Service
sudo systemctl enable --now mysqld
```

---
## 3. Install Required Packages

```bash
# Run this command in all node
sudo dnf install -y mysql-community-server mysql-shell
sudo dnf install -y policycoreutils-python-utils
```

---

## 📁 4. Directory Setup

```bash
# Run this command in all node
sudo mkdir -p /mysql/$(hostname)/datafile /mysql/$(hostname)/log /mysql/$(hostname)/tmp /var/lib/mysql
sudo chown -R mysql:mysql /mysql/ /var/lib/mysql
sudo chmod -R 755 /mysql/ /var/lib/mysql
```

---

## 5. MySQL Configuration (`/etc/my.cnf`)

#### Example: **dc-node1**

```ini
[mysqld]
datadir=/mysql/dc-node1/datafile
log-error=/mysql/dc-node1/log/mysqld.log
tmpdir=/mysql/dc-node1/tmp
report_host=192.168.56.11
socket=/var/lib/mysql/mysql.sock

server_id=1
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE

disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"

loose-group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
loose-group_replication_start_on_boot=off
loose-group_replication_local_address="dc-node1:33061"
loose-group_replication_group_seeds="dc-node1:33061,dc-node2:33061,dc-node3:33061"
```

⚠️ Update accordingly for **dc-node2** and **dc-node3**:

* `server_id`
* `report_host`
* `datadir path`

---

## 🔐6. SELinux Configuration

```bash
# Run this command in all node
sudo semanage fcontext -a -t mysqld_db_t "/mysql/$(hostname)/datafile(/.*)?"
sudo semanage fcontext -a -t mysqld_log_t "/mysql/$(hostname)/log(/.*)?"
sudo semanage fcontext -a -t mysqld_tmp_t "/mysql/$(hostname)/tmp(/.*)?"
sudo semanage fcontext -a -t mysqld_var_run_t "/var/lib/mysql/mysql.sock"

sudo restorecon -Rv /mysql/$(hostname)
sudo restorecon -Rv /var/lib/mysql

# Verify the changes
ls -Zd /mysql/$(hostname)/datafile/ 
ls -Zd /mysql/$(hostname)/log/ 

```

---

## 7. Firewall Configuration
Run these commands on all nodes to allow cluster communication:
```bash
sudo firewall-cmd --permanent --add-port={3306,33060,33061,33062}/tcp
sudo firewall-cmd --reload
```

---

## 8. Initialize MySQL

Run on each node:

```bash
sudo mysqld --initialize --user=mysql --datadir=/mysql/<node>/datafile
```

Example:

```bash
sudo mysqld --initialize --user=mysql --datadir=/mysql/dc-node1/datafile
```

---

## 9. Start MySQL Service In all node

```bash
sudo systemctl start mysqld
sudo systemctl enable mysqld
```

---

## 🔑 10. Retrieve Temporary Password From all Node
Find the temporary root password on each node:
```bash
sudo grep 'temporary password' /mysql/<node>/log/mysqld.log
```
Example:

```bash
sudo grep 'temporary password' /var/log/mysqld.log
```
---

## 🔐 11. Change Root Password In all Node

```bash
mysql -u root -p
```

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourStrongPassword$';
FLUSH PRIVILEGES;
```

---

## ⚡ 12. Configure Cluster (Using MySQL Shell)

### Connect from **dc-node1**

```bash
mysqlsh --uri root@dc-node1:3306
```

Switch to JS mode:

```javascript
\js
```

---

### Configure Instances

```javascript
dba.configureInstance('root@dc-node1:3306')
```

Set:

```
Account Host: 192.168.56.%
```
#### Repeat this in all node make sure to change the command during node two and three and account host is same
---

## 🧩 13. Create Cluster From node-1

```javascript
var cluster = dba.createCluster('WafiCluster');
```

---

## ➕ 14. Add Secondary Nodes

```javascript
cluster.addInstance('root@dc-node2:3306')
cluster.addInstance('root@dc-node3:3306')
```

👉 Select:

```
Recovery Method: Clone (C)
```

---

## 📊 15. Verify Cluster Status

```javascript
cluster.status()
```

---

## ✅ Final Outcome

* 3-node MySQL InnoDB Cluster
* Automatic failover
* Group Replication enabled
* High availability achieved

---

## 📌 Notes

* Ensure **unique `server_id` per node**
---


##  Vagrant Lab Setup (Optional but Recommended)

This section allows you to **quickly spin up the 3-node environment locally** using **Vagrant**.

---

### 🔧 Install Requirements

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients virtinst

# Install Vagrant plugin
vagrant plugin install vagrant-libvirt
```

---

### 📁 Create Vagrantfile

Create a file named `Vagrantfile` in your project directory:

```ruby
Vagrant.configure("2") do |config|

  config.vm.box = "generic/oracle9"

  # Common provider settings
  config.vm.provider :libvirt do |lv|
    lv.driver = "qemu"
  end

  # Node 1
  config.vm.define "dc-node1" do |node|
    node.vm.hostname = "dc-node1"

    node.vm.network "private_network",
                    ip: "192.168.56.11"

    node.vm.provider :libvirt do |lv|
      lv.memory = 2048
      lv.cpus   = 2
    end
  end

  # Node 2
  config.vm.define "dc-node2" do |node|
    node.vm.hostname = "dc-node2"

    node.vm.network "private_network",
                    ip: "192.168.56.12"

    node.vm.provider :libvirt do |lv|
      lv.memory = 2048
      lv.cpus   = 2
    end
  end

  # Node 3
  config.vm.define "dc-node3" do |node|
    node.vm.hostname = "dc-node3"

    node.vm.network "private_network",
                    ip: "192.168.56.13"

    node.vm.provider :libvirt do |lv|
      lv.memory = 2048
      lv.cpus   = 2
    end
  end

end
```

---

### ▶️ Start the Environment

```bash
vagrant up
```

---

### 🔐 Access Nodes

```bash
# Access Node 1 (Primary)
vagrant ssh dc-node1

# Access Node 2
vagrant ssh dc-node2

# Access Node 3
vagrant ssh dc-node3
```

---

### 📊 Verify VMs

```bash
vagrant status
```

---

### 🧹 Destroy Lab (Cleanup)

```bash
vagrant destroy -f
```

---

