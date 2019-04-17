# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative 'vagrant_ros_guest_plugin.rb'

# To enable rsync folder share change to false
$number_of_nodes = 2
$vm_mem = "1024"
$vb_gui = false

$hostname = "node-%02d"
$boximage = "comiq/dockerbox"  #either box-cutter/ubuntu1404-docker or rancherio/rancheros

def config_node(node, hostname, i)
  node.vm.hostname = hostname
  node_ip = "172.19.8.#{i+14}"
  node.vm.network 'private_network', ip: node_ip
  node.vm.provision :shell, :inline => 'curl -sfL https://get.k3s.io | K3S_URL=https://172.19.8.11:6443 K3S_CLUSTER_SECRET=abc123!! sh -', :privileged => true
end

Vagrant.configure(2) do |config|
  config.vm.box_download_insecure = true
  config.vm.box   = $boximage

  config.vm.define 'rancher-01' do |node|
      node.vm.provider "virtualbox" do |vb|
          vb.memory = "2048"
          vb.gui = $vb_gui
      end

      node.vm.hostname = 'rancher-01'
      node_ip = "172.19.8.10"
      node.vm.network 'private_network', ip: node_ip
      node.vm.provision :shell, :inline => 'docker run -d --restart=unless-stopped -v /opt/rancher:/var/lib/rancher -p 80:80 -p 443:443 rancher/rancher', :privileged => true
      
      #should be able to remove
      node.vm.synced_folder ".", "/opt/rancher", type: "rsync",
          rsync__exclude: ".git/", rsync__args: ["--verbose", "--archive", "--delete", "--copy-links"],
          disabled: true

  end


  config.vm.define 'k3s-01' do |node|
      node.vm.provider "virtualbox" do |vb|
          vb.memory = "512"
          vb.gui = $vb_gui
      end

      node.vm.hostname = 'k3s-01'
      node_ip = "172.19.8.11"
      node.vm.network 'private_network', ip: node_ip
      node.vm.provision :shell, :inline => 'curl -sfL https://get.k3s.io | K3S_CLUSTER_SECRET=abc123!! INSTALL_K3S_EXEC="--disable-agent" sh -', :privileged => true
      
      #should be able to remove
      node.vm.synced_folder ".", "/opt/rancher", type: "rsync",
          rsync__exclude: ".git/", rsync__args: ["--verbose", "--archive", "--delete", "--copy-links"],
          disabled: true

  end


  (1..$number_of_nodes).each do |i|
    hostname = $hostname % i

    config.vm.define hostname do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.memory = $vm_mem
            vb.gui = $vb_gui
        end

        config_node(node, hostname, i)

        node.vm.synced_folder ".", "/opt/rancher", type: "rsync",
            rsync__exclude: ".git/", rsync__args: ["--verbose", "--archive", "--delete", "--copy-links"],
            disabled: true

    end
  
  end
  
end
