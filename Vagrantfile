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

	config.vm.define "urbackupserver", autostart: true do | urbck |
		urbck.vm.box = "ubuntu/bionic64"
		urbck.vm.hostname = "urbackupserver"
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
				vb.customize ['createmedium', 'disk', '--filename', dataDisk, '--format', 'VMDK', '--size', 200 * 1024] #200GB
				#vb.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 4]
				vb.customize ['storageattach', :id,  '--storagectl', 'SCSI', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', dataDisk]
			end
		end
		urbck.vm.network "forwarded_port", guest: 55414, host: 55414
		urbck_win10.vm.network "private_network", type: "dhcp"

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
			make install

			apt -y autoremove
			
			#commenting the following line generates a vagrant error on Windows
			echo -e "[Unit]\nDescription=UrBackup Daemon\nAfter=network-online.target\n\n[Service]\nType=simple\nExecStart=/usr/local/bin/urbackupsrv run\nRestart=on-failure\n\n[Install]\nWantedBy=multi-user.target" > /etc/systemd/system/urbackupsrv.service
			systemctl enable urbackupsrv
			systemctl start urbackupsrv

			# Mount data point
			mkdir /data
			chown urbackup: /data
			mkfs.ext4 /dev/sdc
			echo -e '/dev/sdc /data ext4 defaults,auto 0 2' >> /etc/fstab

			if [ -f /var/run/reboot-required ]; then
				reboot
			fi
		SHELL
	end

	##############################
	# Client Side :
	##############################

	config.vm.define "urbackupClientLinux", autostart: false do | urbck_linux |
		urbck_linux.vm.box = "ubuntu/bionic64"
		urbck_linux.vm.hostname = "urbackupClientLinux"
		urbck_linux.vm.box_check_update = true
		urbck_linux.vm.network "private_network", type: "dhcp"
		urbck_linux.vm.provision "shell", inline: <<-SHELL
			apt -y update
			apt -y upgrade
			apt -y dist-upgrade



			apt -y autoremove

			if [ -f /var/run/reboot-required ]; then
				reboot
			fi
		SHELL
	end

	config.vm.define "urbackupClientWin2016", autostart: false do | urbck_win2016 |
		urbck_win2016.vm.box = "mwrock/Windows2016"
		urbck_win2016.vm.hostname = "urbackupClientWin2016"
		urbck_win2016.vm.box_check_update = true
		urbck_win2016.vm.network "private_network", type: "dhcp"
	end

	config.vm.define "urbackupClientWin10", autostart: true do | urbck_win10 |
		urbck_win10.vm.box = "Microsoft/EdgeOnWindows10"
		urbck_win10.vm.hostname = "urbackupClientWin10"
		urbck_win10.vm.box_check_update = true
		urbck_win10.vm.network "private_network", type: "dhcp"
		urbck_win10.vm.guest = :windows
	end

end
