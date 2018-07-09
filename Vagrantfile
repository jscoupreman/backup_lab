# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.synced_folder ".", "/vagrant", disabled: true
	config.vm.provider "virtualbox" do | v |
		v.gui = false
	end
	
	##############################
	# Custom configuration :
	##############################

	##############################
	# Server Side :
	##############################

	config.vm.define "urbackupserver", autostart: true do | urbck_server |
		urbck_server.vm.box = "ubuntu/bionic64"
		urbck_server.vm.hostname = "urbackupserver"
		urbck_server.vm.box_check_update = true

		#line = `VBoxManage list systemproperties | grep "Default machine folder"`
    	#vb_machine_folder = line.split(':')[1].strip()
    	#second_disk = File.join(vb_machine_folder, vb.name, 'disk2.vdi')
		dataDisk = './backup_data.vmdk'

		urbck_server.vm.provider "virtualbox" do | vb |
			#vb.customize ["modifyvm", :id, "--cpus", 4]
			vb.customize ["modifyvm", :id, "--memory", 4096]
			# Building disk files if they don't exist
			if not File.exists?(dataDisk)
				#vb.customize ['createhd', '--filename', dataDisk, '--variant', 'Fixed', '--size', 10 * 1024] #10GB
				vb.customize ['createmedium', 'disk', '--filename', dataDisk, '--format', 'VMDK', '--size', 200 * 1024] #200GB
				#vb.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 4]
				vb.customize ['storageattach', :id,  '--storagectl', 'SCSI', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', dataDisk]
			end
		end
		#urbck_server.vm.network "forwarded_port", guest: 55414, host: 55414
		#urbck_server.vm.network "private_network", type: "dhcp", virtualbox__intnet: true
		urbck_server.vm.network "private_network", ip: "10.0.0.10"
			#echo -e "[Unit]\nDescription=UrBackup Daemon\nAfter=network-online.target\n\n[Service]\nType=simple\nExecStart=/usr/local/bin/urbackupsrv run\nRestart=on-failure\n\n[Install]\nWantedBy=multi-user.target" > /etc/systemd/system/urbackupsrv.service
		
		urbck_server.vm.provision "shell", inline: <<-SHELL
			apt -y update
			apt -y upgrade
			apt -y dist-upgrade

			apt install -y autoconf make g++ zlib1g-dev libcurl4-openssl-dev libcrypto++-dev libfuse-dev pkg-config
			
			apt -y autoremove

			wget https://hndl.urbackup.org/Server/2.2.11/urbackup-server-2.2.11.tar.gz
			tar xf urbackup-server-2.2.11.tar.gz
			cd urbackup-server-2.2.11
			./configure
			make -j 2
			make install

			sed -i "s#/usr/bin/urbackupsrv#$(whereis urbackupsrv | awk '{print $2;}')#g" urbackup-server.service
			cp urbackup-server.service /etc/systemd/system/
			systemctl enable urbackup-server.service
			cp defaults_server /etc/default/urbackupsrv
			cp logrotate_urbackupsrv /etc/logrotate.d/urbackupsrv
			systemctl start urbackup-server


			# Mount data point
			mkdir /data
			mkfs.ext4 /dev/sdc
			echo -e '/dev/sdc /data ext4 defaults,auto 0 2' >> /etc/fstab
			#if [ -f /var/run/reboot-required ]; then
				reboot
				chown urbackup: /data
			#fi
		SHELL
	end

	##############################
	# Client Side :
	##############################

	config.vm.define "urbackupClientLinux", autostart: true do | urbck_linux |
		urbck_linux.vm.box = "ubuntu/bionic64"
		urbck_linux.vm.hostname = "urbackupClientLinux"
		urbck_linux.vm.box_check_update = true
		urbck_linux.vm.network "private_network", ip: "10.0.0.11"

		urbck_linux.vm.provision "shell", inline: <<-SHELL
			apt -y update
			apt -y upgrade
			apt -y dist-upgrade

			apt install -y autoconf make g++ libwxgtk3.0-dev zlib1g-dev libcurl4-openssl-dev libcrypto++-dev libfuse-dev pkg-config
			apt -y autoremove

			wget https://hndl.urbackup.org/Client/2.2.6/urbackup-client-2.2.6.tar.gz
			tar xzf urbackup-client-2.2.6.tar.gz

			cd urbackup-client-2.2.6.0
			./configure
			make -j 2
			make install

			cp urbackupclientbackend-debian.service /etc/systemd/system/
			systemctl enable urbackupclientbackend-debian.service
			cp defaults_client /etc/default/urbackupclient
			cp defaults_server /etc/default/urbackupsrv
			systemctl start urbackupclientbackend-debian

			if [ -f /var/run/reboot-required ]; then
				reboot
			fi
		SHELL
	end

	config.vm.define "urbackupClientWin2012R2", autostart: true do | urbck_win2012R2 |
		urbck_win2012R2.vm.box = "opentable/win-2012r2-standard-amd64-nocm"
		urbck_win2012R2.vm.hostname = "urbackupClientWin2012R2"
		urbck_win2012R2.vm.box_check_update = true
		urbck_win2012R2.vm.network "private_network", ip: "10.0.0.12"
	end

	config.vm.define "urbackupClientWin2016", autostart: true do | urbck_win2016 |
		urbck_win2016.vm.box = "mwrock/Windows2016"
		urbck_win2016.vm.hostname = "urbackupClientWin2016"
		urbck_win2016.vm.box_check_update = true
		urbck_win2016.vm.network "private_network", ip: "10.0.0.13"
	end

	config.vm.define "urbackupClientWin10", autostart: true do | urbck_win10 |
		urbck_win10.vm.box = "Microsoft/EdgeOnWindows10"
		urbck_win10.vm.hostname = "urbackupClientWin10"
		urbck_win10.vm.box_check_update = true
		urbck_win10.vm.network "private_network", ip: "10.0.0.14"
	end

	#config.vm.define "urbackupClientWin7", autostart: false do | urbck_win7 |
	#	urbck_win7.vm.box = "opentable/win-7-professional-amd64-nocm"
	#	urbck_win7.vm.hostname = "urbackupClientWin7"
	#	urbck_win7.vm.box_check_update = true
	#	urbck_win7.vm.network "private_network", ip: "10.0.0.15"
	#end

end
