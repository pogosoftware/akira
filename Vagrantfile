# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"

  config.vm.define "nexus" do |nexus|
    hostname = "nexus.akira.internal"
    dataDisk = "E:\\Virtual Machines Disks\\akira\\nexus-datadisk001.vdi"
    dataDiskSize = 20480

    config.vm.hostname = "#{hostname}"
    config.vm.network "private_network", ip: "192.168.100.200"

    config.vm.provider "virtualbox" do |vb|
      vb.name = "#{hostname}"
      vb.gui = false
      vb.cpus = 2
      vb.memory = 2048

      unless File.exist?("#{dataDisk}")
        vb.customize ['createhd', '--filename', "#{dataDisk}", '--variant', 'Fixed', '--size', "#{dataDiskSize}"]
      end
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--hotpluggable', 'on', '--medium', "#{dataDisk}"]
    end

    nexus.vm.provision "shell", inline: <<-SHELL
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
