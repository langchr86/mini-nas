# -*- mode: ruby -*-
# vi: set ft=ruby :

vm_name = "mini-nas"

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
  config.vm.box_version = "3.0.12"

  config.vm.network "public_network"

  config.ssh.forward_agent = true
  config.ssh.forward_x11 = true

  config.vm.provider "virtualbox" do |vb|
    vb.name = vm_name
    vb.memory = "4096"
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--vram", "64"]
    vb.customize ["modifyvm", :id, "--spec-ctrl", "on"]

    disk_count = 4
    disk_size_gb = 20
    disk_format = 'vdi'
    disk_controller = 'SCSI'

    (0..disk_count-1).each do |i|
	  disk_file = "disks/#{vm_name}-data-disk-#{i}.#{disk_format}"
	  unless File.exist?(disk_file)
	    if (i == 0)
	      vb.customize ["storagectl", :id, "--name", disk_controller, "--add", "sata"]		# create disk controller if not already existing
	    end
        vb.customize ['createhd', '--filename', disk_file, '--format', disk_format, '--size', disk_size_gb * 1024]
      end
	  vb.customize ['storageattach', :id,  '--storagectl', disk_controller, '--port', i+2, '--device', 0, '--type', 'hdd', '--medium', disk_file]
    end

  end

  config.vm.hostname = vm_name
  config.vm.synced_folder "./ansible/", "/vagrant/ansible"

  # remove not yet supported ansible ppa
  config.vm.provision "shell",
    inline: "rm -f /etc/apt/sources.list.d/ansible-ansible-focal.list"

  config.vm.provision "ansible_local" do |ansible|
    ansible.install_mode = "pip"
    ansible.playbook = "/vagrant/ansible/playbook.yml"
    ansible.galaxy_role_file = "/vagrant/ansible/requirements.yml"
  end

end
