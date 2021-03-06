# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"

  (0..2).each do |index|
    config.vm.define "controller#{index}" do |controller|
      hostname = "controller#{index}.akira.internal"
      dataDisk = "E:\\Virtual Machines Disks\\akira\\controller#{index}-datadisk001.vdi"
      dataDiskSize = 10240

      controller.vm.hostname = "#{hostname}"
      controller.vm.network "private_network", ip: "10.240.0.1#{index}", virtualbox__intnet: "kubernetes"
  
      controller.vm.provider "virtualbox" do |vb|
        vb.name = "#{hostname}"
        vb.gui = false
        vb.cpus = 1
        vb.memory = 1024

        unless File.exist?("#{dataDisk}")
          vb.customize ['createhd', '--filename', "#{dataDisk}", '--variant', 'Fixed', '--size', "#{dataDiskSize}"]
        end
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--hotpluggable', 'on', '--medium', "#{dataDisk}"]
      end
        
      controller.vm.provision "file", source: "./certs/controller.akira.internal-key.pem", destination: "/tmp/etcd.key"
      controller.vm.provision "file", source: "./certs/controller.akira.internal.pem", destination: "/tmp/etcd.pem"
      controller.vm.provision "file", source: "./certs/akira-ca.pem", destination: "/tmp/ca.pem"
  
      controller.vm.provision "shell", inline: <<-SHELL
        mkdir -p /srv/ssl/{private,certs}/etcd /srv/ssl/ca
  
        mv /tmp/etcd.key /srv/ssl/private/etcd
        mv /tmp/etcd.pem /srv/ssl/certs/etcd
        mv /tmp/ca.pem /srv/ssl/ca
  
        chmod 644 /srv/ssl/ca/ca.pem /srv/ssl/private/etcd/etcd.key /srv/ssl/certs/etcd/etcd.pem
  
        apt-get -qq update
      SHELL
  
      controller.vm.provision "ansible_local" do |ansible|
        ansible.become = true
        ansible.playbook = "ansible/playbook.yml"
        ansible.galaxy_role_file = "ansible/requirements.yml"
        ansible.galaxy_roles_path = "/etc/ansible/roles"
        ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
      end
    end
  end

  

  config.vm.define "manager" do |manager|
    hostname = "manager.akira.internal"

    manager.vm.hostname = "#{hostname}"
    manager.vm.network "private_network", ip: "192.168.100.200"

    manager.vm.provider "virtualbox" do |vb|
      vb.name = "#{hostname}"
      vb.gui = false
      vb.cpus = 1
      vb.memory = 1024
    end

    manager.vm.provision "ansible_local" do |ansible|
      ansible.become = true
      ansible.playbook = "ansible/playbook.yml"
      ansible.galaxy_role_file = "ansible/requirements.yml"
      ansible.galaxy_roles_path = "/etc/ansible/roles"
      ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
    end
  end

  config.vm.define "vault" do |vault|
    hostname = "vault.akira.internal"
    dataDisk = "E:\\Virtual Machines Disks\\akira\\vault-datadisk001.vdi"
    dataDiskSize = 10240

    vault.vm.hostname = "#{hostname}"
    vault.vm.network "private_network", ip: "192.168.100.201"

    vault.vm.provider "virtualbox" do |vb|
      vb.name = "#{hostname}"
      vb.gui = false
      vb.cpus = 1
      vb.memory = 1024

      unless File.exist?("#{dataDisk}")
        vb.customize ['createhd', '--filename', "#{dataDisk}", '--variant', 'Fixed', '--size', "#{dataDiskSize}"]
      end
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--hotpluggable', 'on', '--medium', "#{dataDisk}"]
    end

    vault.vm.provision "file", source: "./certs/vault-server.akira.internal-key.pem", destination: "/tmp/server.key"
    vault.vm.provision "file", source: "./certs/vault-server.akira.internal.pem", destination: "/tmp/server.pem"
    vault.vm.provision "file", source: "./certs/vault-client.akira.internal-key.pem", destination: "/tmp/client.key"
    vault.vm.provision "file", source: "./certs/vault-client.akira.internal.pem", destination: "/tmp/client.pem"
    vault.vm.provision "file", source: "./certs/akira-ca.pem", destination: "/tmp/ca.pem"

    vault.vm.provision "shell", inline: <<-SHELL
      mkdir -p /srv/ssl/{private,certs}/vault /srv/ssl/ca

      mv /tmp/{server,client}.key /srv/ssl/private/vault
      mv /tmp/{server,client}.pem /srv/ssl/certs/vault
      mv /tmp/ca.pem /srv/ssl/ca

      chmod 644 /srv/ssl/ca/ca.pem /srv/ssl/private/vault/{client,server}.key /srv/ssl/certs/vault/{client,server}.pem

      apt-get -qq update
      apt-get install -y unzip
    SHELL

    vault.vm.provision "ansible_local" do |ansible|
      ansible.become = true
      ansible.playbook = "ansible/playbook.yml"
      ansible.galaxy_role_file = "ansible/requirements.yml"
      ansible.galaxy_roles_path = "/etc/ansible/roles"
      ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
    end
  end

  config.vm.define "nexus" do |nexus|
    hostname = "nexus.akira.internal"
    dataDisk = "E:\\Virtual Machines Disks\\akira\\nexus-datadisk001.vdi"
    dataDiskSize = 20480

    nexus.vm.hostname = "#{hostname}"
    nexus.vm.network "private_network", ip: "192.168.100.202"

    nexus.vm.provider "virtualbox" do |vb|
      vb.name = "#{hostname}"
      vb.gui = false
      vb.cpus = 2
      vb.memory = 2048

      unless File.exist?("#{dataDisk}")
        vb.customize ['createhd', '--filename', "#{dataDisk}", '--variant', 'Fixed', '--size', "#{dataDiskSize}"]
      end
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--hotpluggable', 'on', '--medium', "#{dataDisk}"]
    end

    nexus.vm.provision "file", source: "./certs/nexus-server.akira.internal-key.pem", destination: "/tmp/server.key"
    nexus.vm.provision "file", source: "./certs/nexus-server.akira.internal.pem", destination: "/tmp/server.pem"
    nexus.vm.provision "file", source: "./certs/akira-ca.pem", destination: "/tmp/ca.pem"

    nexus.vm.provision "shell", inline: <<-SHELL
      mkdir -p /srv/ssl/{private,certs}/nexus /srv/ssl/ca

      mv /tmp/server.key /srv/ssl/private/nexus
      mv /tmp/server.pem /srv/ssl/certs/nexus
      mv /tmp/ca.pem /srv/ssl/ca

      chmod 644 /srv/ssl/ca/ca.pem /srv/ssl/private/nexus/server.key /srv/ssl/certs/nexus/server.pem

      apt-get -qq update
      apt-get install -y openjdk-8-jre-headless python-pip apache2
      pip install jmespath
      a2enmod ssl rewrite proxy proxy_http headers
      systemctl restart apache2
    SHELL

    nexus.vm.provision "ansible_local" do |ansible|
      ansible.become = true
      ansible.playbook = "ansible/playbook.yml"
      ansible.galaxy_role_file = "ansible/requirements.yml"
      ansible.galaxy_roles_path = "/etc/ansible/roles"
      ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
    end
  end
end
