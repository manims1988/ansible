# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'no'

Vagrant.configure("2") do |config|
  config.vbguest.auto_update = false
  config.hostmanager.enabled = true
  NodeCount = 1
    (1..NodeCount).each do |i|
      config.vm.define "app#{i}" do |appvm|
        appvm.vm.box = "centos/7"
        appvm.vm.hostname = "app#{i}.example.com"
        appvm.vm.network "private_network", ip: "192.168.56.10#{i}"
        appvm.vm.provider "virtualbox" do |va|
          va.name = "app#{i}"
        end
        appvm.vm.provision "shell", inline: <<-SHELL
          sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
          systemctl reload sshd
        SHELL
      end
  end

  config.vm.define 'db1' do |dbvm|
    dbvm.vm.box = 'centos/7'
    dbvm.vm.provider 'virtualbox' do |vc|
      vc.name = 'db1'
    end
    dbvm.vm.hostname = 'db1.example.com'
    dbvm.vm.network 'private_network', ip: '192.168.56.102'
    dbvm.vm.provision 'shell', inline: <<-SHELL
      sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
      systemctl reload sshd
    SHELL
  end

  config.vm.define 'master' do |master|
    master.vm.box = "centos/7"
    master.vm.provider "virtualbox" do |vb|
      vb.name = 'master'
    end
    master.vm.hostname = 'master'
    master.vm.network :private_network, ip: '192.168.56.110'
    master.vm.provision "shell", inline: <<-SHELL
      echo '[Task-1] EPEL repository installation'
      yum makecache
      yum install -y epel-release
      echo '[Task-2] Ansible installation'
      yum makecache
      yum install ansible -y
      echo '[Task-3] Check Ansible Version'
      ansible --version
      echo '[Task-4] Enable password authentication'
      sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
      systemctl reload sshd
      echo '[Task-5] Generate Private and Public Key'
      ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa <<< y
      echo '[Task-6] Copy the Key on all 3 VM'
      sshpass -p vagrant ssh-copy-id -o StrictHostKeyChecking=no root@192.168.56.101
      sshpass -p vagrant ssh-copy-id -o StrictHostKeyChecking=no root@192.168.56.102
      echo '[Task-7] Appending Ansible host inventory file'
      echo '[app]' >> /etc/ansible/hosts
      echo '192.168.56.101' >> /etc/ansible/hosts
      echo '[db]' >> /etc/ansible/hosts
      echo '192.168.56.102' >> /etc/ansible/hosts
      echo '[server]' >> /etc/ansible/hosts
      echo '192.168.56.102' >> /etc/ansible/hosts
      echo '192.168.56.101' >> /etc/ansible/hosts
      echo '[Task-8] Ansible check'
      ansible all -m ping --one-line
      echo '[Task-9] jenkins install'
      yum update -y
      yum install wget epel-release java-11-openjdk -y
      wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
      rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
      yum install jenkins -y
      systemctl daemon-reload
      systemctl enable jenkins
      systemctl start jenkins
      systemctl status jenkins
    SHELL
  end
end