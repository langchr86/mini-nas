# -*- mode: ruby -*-
# vi: set ft=ruby :

vm_name = "mini-nas"

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.box_version = "20200407.0.0"

  # config.vm.network "private_network", type: "dhcp"

  config.ssh.forward_agent = true
  config.ssh.forward_x11 = true

  config.vm.provider "virtualbox" do |vb|
    vb.name = vm_name
    vb.memory = "4096"
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--vram", "64"]
    vb.customize ["modifyvm", :id, "--spec-ctrl", "on"]

    disk_count = 4
    disk_size_gb = 50
	disk_format = 'vdi'
	disk_controller = 'SCSI'		# already existing in ubuntu box
    (0..disk_count-1).each do |i|
	  disk_file = "disks/#{vm_name}-data-disk-#{i}.#{disk_format}"
	  unless File.exist?(disk_file)
        vb.customize ['createhd', '--filename', disk_file, '--format', disk_format, '--size', disk_size_gb * 1024]
      end
	  vb.customize ['storageattach', :id,  '--storagectl', disk_controller, '--port', i+2, '--device', 0, '--type', 'hdd', '--medium', disk_file]
    end

  end

  config.vm.hostname = vm_name

  #config.vm.provision "ansible_local" do |ansible|
  #  ansible.playbook = "/vagrant/ansible/playbook-headless.yml"
  #  ansible.galaxy_role_file = "/vagrant/ansible/requirements.yml"
  #end

end
