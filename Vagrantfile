Vagrant.configure("2") do |config|

  # Default Box: Oracle Linux 9
  config.vm.box = "generic/oracle9"

  # Node 1 Configuration
  config.vm.define "dc-node1" do |node|
    node.vm.hostname = "dc-node1"
    node.vm.network "private_network", ip: "192.168.56.11"
    
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
      vb.name = "dc-node1-WafiCluster"
    end
  end

  # Node 2 Configuration
  config.vm.define "dc-node2" do |node|
    node.vm.hostname = "dc-node2"
    node.vm.network "private_network", ip: "192.168.56.12"
    
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
      vb.name = "dc-node2-WafiCluster"
    end
  end

  # Node 3 Configuration
  config.vm.define "dc-node3" do |node|
    node.vm.hostname = "dc-node3"
    node.vm.network "private_network", ip: "192.168.56.13"
    
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
      vb.name = "dc-node3-WafiCluster"
    end
  end

end
