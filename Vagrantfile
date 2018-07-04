# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.provider "virtualbox" do | v |
		v.gui = false
	end
	
	##############################
	# Custom configuration :
	##############################
	
	config.vm.define "backuppc", autostart: false do | bpc |
		bpc.vm.box = "ubuntu/bionic64"
		bpc.vm.hostname = "backuppc"
		bpc.vm.box_check_update = true
		bpc.vm.provider "virtualbox" do | v |
			v.customize ["modifyvm", :id, "--cpus", 6]
			v.customize ["modifyvm", :id, "--memory", 8196]
		end
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
	
	config.vm.define "urbackup", autostart: true do | urbck |
		urbck.vm.box = "ubuntu/bionic64"
		urbck.vm.hostname = "UrBackup"
		urbck.vm.box_check_update = true
		urbck.vm.provider "virtualbox" do | v |
			v.customize ["modifyvm", :id, "--cpus", 4]
			v.customize ["modifyvm", :id, "--memory", 8196]
		end
		urbck.vm.network "forwarded_port", guest: 55414, host: 55414
		urbck.vm.provision "shell", inline: <<-SHELL
			apt -y update
			apt -y upgrade
			apt -y dist-upgrade
			
			apt install -y autoconf make g++ zlib1g-dev libcurl4-openssl-dev libcrypto++-dev libfuse-dev pkg-config
			git clone https://github.com/uroni/urbackup_backend.git
			cd urbackup_backend
			./switch_build.sh server
			autoreconf --install
			./configure
			make -j 4
			sudo make install

			apt -y autoremove
			
			echo -e "[Unit]\nDescription=UrBackup Daemon\nAfter=network-online.target\n\n[Service]\nType=simple\nExecStart=/usr/local/bin/urbackupsrv run\nRestart=on-failure\n\n[Install]\nWantedBy=multi-user.target" > /etc/systemd/system/urbackupsrv.service
			systemctl enable urbackupsrv
			systemctl start urbackupsrv

			if [ -f /var/run/reboot-required ]; then
				reboot
			fi
		SHELL
	end	
end
