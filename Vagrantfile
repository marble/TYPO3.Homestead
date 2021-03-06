# -*- mode: ruby -*-
# vi: set ft=ruby :

# Examples:
# https://gitlab.com/gitlab-org/cookbook-gitlab/blob/master/Vagrantfile#L80
# http://www.tomaz.me/2013/10/14/solution-for-ansible-git-module-getting-stuck-on-clone.html

# You can ask for more memory and cores when creating your Vagrant machine:
# VAGRANT_MEMORY=2048 VAGRANT_CORES=4 vagrant up
MEMORY = ENV['VAGRANT_MEMORY'] || 2048
CORES = ENV['VAGRANT_CORES'] || 1

# Network
PRIVATE_NETWORK = ENV['VAGRANT_PRIVATE_NETWORK'] || '192.168.12.12'
HOSTNAME = ENV['VAGRANT_HOSTNAME'] || 'typo3.homestead'

# Determine if we need to forward ports
FORWARD = ENV['VAGRANT_FORWARD'] || 1

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = 2

# Boot the box with the gui enabled
DEBUG = ENV['VAGRANT_DEBUG'] || false

# Generate SSH keys for these known hosts
# knownHosts = [ 'github.com', 'git.typo3.org' ]

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	config.vm.hostname = HOSTNAME
	config.vm.box = "ubuntu/trusty64"
	config.vm.boot_timeout = 120
	config.hostsupdater.aliases = [
		'4.5.typo3.cms',
		'4.5.39.typo3.cms',
		'6.2.typo3.cms',
		'6.2.9.typo3.cms',
		'7.0.typo3.cms',
		'7.0.2.typo3.cms',
		'1.2.typo3.neos',
		'dev-master.typo3.neos'
		]

	# Network
	config.vm.network :private_network, ip: PRIVATE_NETWORK
	if FORWARD.to_i > 0
		config.vm.network :forwarded_port, guest: 80, host: 8080
		config.vm.network :forwarded_port, guest: 3306, host: 33060
		config.vm.network :forwarded_port, guest: 5432, host: 54320
		config.vm.network :forwarded_port, guest: 35729, host: 35729
	end

	# SSH
	config.ssh.forward_agent = true
	config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'" # avoids 'stdin: is not a tty' error.
	config.ssh.forward_agent = true
	# 	config.vm.provision "shell", inline: "echo -e '#{File.read("#{Dir.home}/.ssh/id_rsa")}' > '/home/vagrant/.ssh/id_rsa'"
# 	config.ssh.username = "root"
# 	config.ssh.private_key_path = "phusion.key"

	# Virtualbox
	config.vm.provider :virtualbox do |v|
		v.gui = !!DEBUG
		v.cpus = CORES.to_i
		v.memory = MEMORY.to_i
		v.customize [
			"modifyvm", :id,
			"--chipset", "ich9",
			"--cpuexecutioncap", "90",
			"--natdnshostresolver1", "on"
			]

		if CORES.to_i > 1
			v.customize [
				"modifyvm", :id,
				"--chipset", "ich9",
				"--cpuexecutioncap", "90",
				"--ioapic", "on",
				"--natdnshostresolver1", "on"
				]
		end
	end

	# Vmware Fusion
	config.vm.provider :vmware_fusion do |v, override|
		override.vm.box = "trusty64_fusion"
		override.vm.box_url = "http://files.vagrantup.com/trusty64_vmware_fusion.box"
		v.vmx["memsize"] = MEMORY.to_i
		v.vmx["numvcpus"] = CORES.to_i
	end

	# Parallels
	config.vm.provider :parallels do |v, override|
		v.customize ["set", :id, "--memsize", MEMORY, "--cpus", CORES]
	end

	# Ansible | http://docs.ansible.com/playbooks_best_practices.html
	config.vm.provision "ansible" do |ansible|
# 		ansible.verbose = "v"
		ansible.playbook = "site.yml"
		ansible.limit = "all"
		ansible.raw_arguments = ENV['ANSIBLE_ARGS']
		ansible.extra_vars = {
			ansible_ssh_user: 'vagrant',
			hostname: HOSTNAME
		}
	end

	# Synced Folders
	config.vm.synced_folder ".", "/vagrant", disabled: true
	config.vm.synced_folder "~/Projects/TYPO3/Development", "/var/www", create: true, group: "www-data", owner: "vagrant", mount_options: ["dmode=775,fmode=664"]
# 	config.vm.synced_folder "~/Projects/DonationBasedHosting", "/var/www", group: "www-data", mount_options: ["dmode=775,fmode=664"]
#mount_options: ["umask=0002,dmask=0002,fmask=0002"]

  # Set no_root_squash to prevent NFS permissions errors on Linux during
  # provisioning, and maproot=0:0 to correctly map the guest root user.
#   if (/darwin/ =~ RUBY_PLATFORM) != nil
#     config.vm.synced_folder "./www", "/var/www", type: "nfs", :bsd__nfs_options => ["maproot=0:0"]
#   else
#     config.vm.synced_folder "./www", "/var/www", type: "nfs", :linux__nfs_options => ["no_root_squash"]
#   end

  	# Use NFS for the shared folder
# 	config.nfs.map_uid = 1000
# 	config.nfs.map_gid = 33
#  	config.vm.synced_folder '~/Projects/DonationBasedHosting', '/var/www2',
# 		id: 'core',
# 		:nfs => true,
# 		:mount_options => ['rw,nolock,noatime'],
# 		:bsd__nfs_options => ['no_root_squash,maproot=0:0'],
# 		:map_uid => 0,
# 		:map_gid => 0,
# 		:export_options => ['async,insecure,no_subtree_check,no_acl,no_root_squash,noatime']
end
