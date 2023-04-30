# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # generate a token for the cluster
  #token = rand(36**6).to_s(36) + "." + rand(36**16).to_s(36)
  token = (36**5 + rand(36**6)).to_s(36) + "." + (36**15 + rand(36**16)).to_s(36)
  ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip

  config.vm.box = "ubuntu/focal64"
  config.vm.box_check_update = false

  (0..2).each do |i|
    config.vm.define "k8s-#{i}" do |node|
      node.vm.hostname = "k8s-#{i}"
      node.vm.provider "virtualbox" do |v|
        v.memory = 4096
        v.cpus = 4
      end
      last_ip = i + 4
      node.vm.network "private_network", ip: "192.168.56.#{last_ip}"
      node.vm.network "forwarded_port", guest: 4646, host: 14646 + i, guest_ip: "192.168.56.#{last_ip}"
      node.vm.provision "shell", inline: <<-SHELL
        sudo swapoff -a
	      sudo sed -i 's/.*swap.*//g' /etc/fstab
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
      SHELL
    end
  end
end
