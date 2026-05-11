MySQL 8.4 LTS InnoDB Cluster Setup (WafiCluster)This repository documents the step-by-step implementation of a Highly Available (HA) MySQL InnoDB Cluster using Group Replication and MySQL Router.🏗️ Cluster ArchitectureThe architecture consists of a three-node cluster providing automatic failover and a MySQL Router acting as the intelligent gateway.                     ┌────────────────────────┐
                     │      MySQL Router      │
                     │  (Port 6446 - R/W)     │
                     └──────────┬─────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
   ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
   │   dc-node1   │      │   dc-node2   │      │   dc-node3   │
   │ 192.168.56.11│      │ 192.168.56.12│      │ 192.168.56.13│
   │   PRIMARY    │◄────►│   SECONDARY  │◄────►│   SECONDARY  │
   └──────────────┘      └──────────────┘      └──────────────┘
          │                     │                     │
          └─────────── Group Replication ─────────────┘
                       (Port 33061)
🖥️ Node DetailsNode NameIP AddressRoleOSdc-node1192.168.56.11PrimaryOracle Linux 9dc-node2192.168.56.12SecondaryOracle Linux 9dc-node3192.168.56.13SecondaryOracle Linux 9🌐 1. Network & Firewall ConfigurationTo ensure the nodes can communicate for Group Replication and Cloning, run this on all nodes:Bash# Set up firewall ports
sudo firewall-cmd --permanent --add-port={3306,33060,33061,33062}/tcp
sudo firewall-cmd --reload

# Update /etc/hosts for easy naming
sudo vi /etc/hosts
# Add:
192.168.56.11 dc-node1
192.168.56.12 dc-node2
192.168.56.13 dc-node3
📦 2. Installation (All Nodes)Install the MySQL 8.4 LTS repository and the required server and shell packages:Bashsudo dnf install -y https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql-community-server mysql-shell
sudo systemctl enable --now mysqld
⚡ 3. Cluster Initialization (Using MySQL Shell)Connect to the Seed Node (dc-node1)Bashmysqlsh --uri root@192.168.56.11:3306
Configure and Create ClusterWithin the MySQL Shell (JS > mode):JavaScript// Configure the instance for cluster usage
dba.configureInstance('root@192.168.56.11:3306')

// Create the cluster
var cluster = dba.createCluster('WafiCluster');
➕ 4. Expanding the ClusterAdd the remaining nodes. When prompted, select [C]lone to synchronize data automatically.JavaScriptcluster.addInstance('root@192.168.56.12:3306')
cluster.addInstance('root@192.168.56.13:3306')
🚦 5. MySQL Router ConfigurationInstall and bootstrap the router on dc-node1 to handle application traffic and Read/Write splitting.Bash# Install Router
sudo yum install mysql-router-community -y

# Bootstrap against the cluster
sudo mysqlrouter --bootstrap root@192.168.56.11:3306 --user=mysqlrouter

# Start the service
sudo systemctl enable --now mysqlrouter
📊 6. Verification & MonitoringCheck Cluster StatusJavaScriptcluster.status()
Verify via SQLSQLSELECT * FROM mysql_innodb_cluster_metadata.instances;
✅ Features AchievedHigh Availability: Automatic failover if the Primary node fails.Fault Tolerance: The cluster maintains quorum with 3 nodes.Read/Write Splitting:Port 6446: Read/Write (Primary)Port 6447: Read-Only (Secondaries)Zero-Manual Recovery: Used MySQL Clone for seamless node synchronization.🛠️ Vagrant Lab EnvironmentYou can recreate this environment using the following Vagrantfile snippet:RubyVagrant.configure("2") do |config|
  config.vm.box = "generic/oracle9"
  
  (1..3).each do |i|
    config.vm.define "dc-node#{i}" do |node|
      node.vm.hostname = "dc-node#{i}"
      node.vm.network "private_network", ip: "192.168.56.1#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
      end
    end
  end
end
