
Vagrant.configure(2) do |configs|

	Dir.chdir('boxes')
	boxIds = Dir.glob('*').select { |boxId| File.exist? "#{boxId}/box.json" }
	Dir.chdir('..')

	boxIds.each {
		|boxId|
		box = JSON.parse(IO.read("./boxes/#{boxId}/box.json", encoding: "utf-8"))

		configs.vm.define boxId, autostart: box["autostart"] do |config|
			config.vm.box = "ubuntu/xenial64"
			config.vm.box_check_update = false

			config.vm.provider "virtualbox" do |vb|
				vb.name = box["title"]
				vb.memory = box["memory"]
			end

			# @warning Quickhack to detect if the box has already been fixed:
			# http://stackoverflow.com/questions/24855635/check-if-vagrant-provisioning-has-been-done
			if File.exist?(".vagrant/machines/#{boxId}/virtualbox/action_provision")

				# box["ports"].each do |port|
				# 	config.vm.network "forwarded_port", guest: port["guest"], host: port["host"]
				# end

				if File.directory?("./boxes/#{boxId}")
					config.vm.synced_folder "./boxes/#{boxId}", "/setup", create: true, owner: 'ubuntu', group: 'www-data'
				end

				# box["folders"].each do |folder|
				# 	hostPath = folder["host"]
				# 	guestPath = folder["guest"]
				# 	puts ">>>>>> TO MOUNT: ./boxes/#{boxId}/#{hostPath} TO #{guestPath}"
				# 	# config.vm.synced_folder "./boxes/#{boxId}/#{hostPath}", folder["guest"], create: true, owner: folder["owner"], group: folder["group"]
				# end

				# box["provision"].each do |provisionId|
				# 	config.vm.provision "shell", name: provisionId, privileged: false, path: "provision/#{provisionId}.sh"
				# end
			else

				config.vm.provision "shell", name: "Adds the missing host", privileged: true, :inline => <<-SHELL
					echo "127.0.0.1 ubuntu-xenial" >> /etc/hosts
				SHELL

				config.vm.provision "shell", name: "Installs the Virtualbox Guest Additions", privileged: false, :inline => <<-SHELL
					sudo apt-get install -y build-essential linux-headers-generic linux-headers-`uname -r`
					cd /tmp
					wget http://download.virtualbox.org/virtualbox/5.0.24/VBoxGuestAdditions_5.0.24.iso -nv
					sudo mount -o loop,ro VBoxGuestAdditions_5.0.24.iso /mnt
					sudo /mnt/VBoxLinuxAdditions.run --nox11
					sudo umount /mnt
					rm VBoxGuestAdditions_5.0.24.iso
				SHELL
			end
		end
	}
end
