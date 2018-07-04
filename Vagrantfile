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


		bpc.vm.provider "virtualbox" do | vb |
			vb.customize ["modifyvm", :id, "--cpus", 6]
			vb.customize ["modifyvm", :id, "--memory", 8196]
			
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

		#line = `VBoxManage list systemproperties | grep "Default machine folder"`
    	#vb_machine_folder = line.split(':')[1].strip()
    	#second_disk = File.join(vb_machine_folder, vb.name, 'disk2.vdi')
		dataDisk = './backup_data.vmdk'

		urbck.vm.provider "virtualbox" do | vb |
			vb.customize ["modifyvm", :id, "--cpus", 4]
			vb.customize ["modifyvm", :id, "--memory", 8196]
			# Building disk files if they don't exist
			if not File.exists?(dataDisk)
				#vb.customize ['createhd', '--filename', dataDisk, '--variant', 'Fixed', '--size', 10 * 1024] #10GB
				vb.customize ['createmedium', 'disk', '--filename', dataDisk, '--format', 'VMDK', '--size', 10 * 1024] #10GB
				#vb.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 4]
				vb.customize ['storageattach', :id,  '--storagectl', 'SCSI', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', dataDisk]
			end
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
