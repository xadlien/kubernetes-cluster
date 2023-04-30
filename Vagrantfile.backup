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
      node.vm.provision "file", source: "./daemon.json", destination: "/tmp/daemon.json"
      node.vm.provision "shell", inline: <<-SHELL
        sudo swapoff -a
	      sudo sed -i 's/.*swap.*//g' /etc/fstab
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl \
          docker docker-compose
        sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
        echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo apt-mark hold kubelet kubeadm kubectl
        sudo mv /tmp/daemon.json /etc/docker/daemon.json
        sudo systemctl restart docker
        curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
        sudo apt-get install apt-transport-https --yes
        echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
        sudo apt-get update
        sudo apt-get install helm
      SHELL
      if i == 0 
        node.vm.provision "shell", inline: "kubeadm init --token #{token} --apiserver-advertise-address 192.168.56.4 --pod-network-cidr 10.244.0.0/16"
        node.vm.provision "shell", inline: "kubectl --kubeconfig /etc/kubernetes/admin.conf label node k8s-0 node-role.kubernetes.io/etcd=etcd"
        node.vm.provision "shell", inline: "kubectl --kubeconfig /etc/kubernetes/admin.conf create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml"
        node.vm.provision "shell", inline: "mkdir /home/vagrant/.kube"
        node.vm.provision "shell", inline: "cp /etc/kubernetes/admin.conf /home/vagrant/.kube/config"
        node.vm.provision "shell", inline: "chown -R vagrant /home/vagrant"
      else 
        node.vm.provision "shell", inline: "kubeadm join 192.168.56.4:6443 --token #{token} --discovery-token-unsafe-skip-ca-verification"
        node.vm.provision "shell", inline: "mkdir /home/vagrant/.kube"
        node.vm.provision "shell", inline: "cp /etc/kubernetes/kubelet.conf /home/vagrant/.kube/config"
        node.vm.provision "shell", inline: "chown -R vagrant /home/vagrant"
      end
    end
  end
end
