# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/bionic64"
  config.vm.provision "shell", inline: <<-SHELL
       which python || (apt-get update; apt-get install -y python)
  SHELL

  (1..2).each do |i|
    config.vm.define "ns#{i}" do |node|
      node.vm.hostname = "ns#{i}.example.com"
      node.vm.network "public_network", ip: "10.1.19.1#{i}", :bridge => "en0: Wi-Fi (AirPort)"
      node.vm.synced_folder "./", "/vagrant", owner: "root", group: "root"
      node.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--memory", 512]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        # set timesync parameters to keep the clocks better in sync
        # sync time every 10 seconds
        vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-interval", 10000 ]
        # adjustments if drift > 100 ms
        vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-min-adjust", 100 ]
        # sync time on restore
        vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-on-restore", 1 ]
        # sync time on start
        vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-start", 1 ]
        # at 1 second drift, the time will be set and not "smoothly" adjusted
        vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 1000 ]
      end
      node.vm.provision "ansible" do |ansible|
        ansible.playbook = 'provisioning/ns.yml'
        ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" }
        ansible.become = true
      end
    end
  end

  config.vm.define :c1 do |node|
    node.vm.hostname = "c1.example.com"
    node.vm.network "public_network", ip: "10.1.19.13", :bridge => "en0: Wi-Fi (AirPort)"
    node.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", 512]
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      # set timesync parameters to keep the clocks better in sync
      # sync time every 10 seconds
      vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-interval", 10000 ]
      # adjustments if drift > 100 ms
      vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-min-adjust", 100 ]
      # sync time on restore
      vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-on-restore", 1 ]
      # sync time on start
      vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-start", 1 ]
      # at 1 second drift, the time will be set and not "smoothly" adjusted
      vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 1000 ]
    end
    node.vm.provision "ansible" do |ansible|
      ansible.playbook = 'provisioning/c.yml'
      ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" }
      ansible.become = true
    end
  end
end
