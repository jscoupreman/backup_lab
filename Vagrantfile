# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.provider "virtualbox" do | v |
		v.gui = false
	end
	
	##############################
	# Custom configuration :
	##############################
	
	config.vm.define "backuppc", autostart: true do | bpc |
		bpc.vm.box = "ubuntu/bionic64"
		bpc.vm.hostname = "backuppc"
		bpc.vm.box_check_update = true
		bpc.vm.provision "shell", inline: <<-SHELL
			apt -y update
			apt -y upgrade
			apt -y dist-upgrade
			
			# apt -y install backuppc

			apt -y autoremove
			
			if [ -f /var/run/reboot-required ]; then
				reboot
			fi
		SHELL
	end
end
