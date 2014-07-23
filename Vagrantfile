# -*- mode: ruby -*-
# vi: set ft=ruby :

# Find the current vagrant directory.
vagrant_dir = File.expand_path(File.dirname(__FILE__))

# Include config from vlad/settings.yml
require 'yaml'
vconfig = YAML::load_file(vagrant_dir + "/settings.yml")

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Configure virtual machine options.
  config.vm.box = vconfig['box']
  config.vm.hostname = vconfig['boxname']
  config.vm.box_check_update = false

  config.vm.network :private_network, ip: vconfig['boxipaddress']

  # Allow caching to be used (see the vagrant-cachier plugin)
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.synced_folder_opts = { 
        type: :nfs,
        mount_options: ['rw', 'vers=3', 'tcp', 'nolock'] 
    }
    config.cache.scope = :box
    config.cache.auto_detect = true
  end

  # Manage vbguest update
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
    config.vbguest.no_remote = true
  end

  # Configure virtual machine setup.
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--memory", 2048]
  end

  # Set up NFS drive.
  nfs_setting = RUBY_PLATFORM =~ /darwin/ || RUBY_PLATFORM =~ /linux/

  # Setup synced folder for site files
  if vconfig['host_synced_folder'] != ''
  config.vm.synced_folder vconfig['host_synced_folder'], "/var/www/site/docroot", :nfs => nfs_setting, create: true, id: "vagrant-webroot", :mount_options => ['nolock,vers=3,udp']
  end

  # SSH Set up.
  config.ssh.forward_agent = true

  # Provision vagrant box with Ansible.
  config.vm.provision 'ansible' do |ansible|
    ansible.playbook = vagrant_dir + '/playbooks/site.yml'
    ansible.extra_vars = { ansible_ssh_user: 'vagrant' }
    ansible.host_key_checking = false
    #ansible.sudo = true    
    #ansible.raw_arguments = ['-v']
    ansible.raw_ssh_args = '-o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -o PasswordAuthentication=no -o IdentitiesOnly=yes'
    if vconfig['ansible_verbosity'] != ''
      ansible.verbose = vconfig['ansible_verbosity']
    end
  end
end
