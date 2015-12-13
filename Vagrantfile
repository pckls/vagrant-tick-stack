# -*- mode: ruby -*-
# # vi: set ft=ruby :

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version ">= 1.7.0"
VAGRANTFILE_API_VERSION = "2"

# Require YAML module
require 'yaml'

# Read YAML file with box details
NODES = YAML.load_file('nodes.yaml')

# Automatically manage plugins
REQUIRED_PLUGINS = %w( vagrant-hosts vagrant-hostmanager )
REQUIRED_PLUGINS.each do |plugin|
    system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
end

DOMAIN = ".vagrant.pckls.io"

# Do the needful
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    NODES.each do |name, data|

        config.vm.define name do |node|

            # Provider specific stuff
            node.vm.provider :virtualbox do |vb|
                vb.cpus = data["cpus"]
                vb.memory = data["memory"]
                vb.name = name
            end

            # Box stuff
            node.vm.box = data["boxname"]
            node.vm.box_url = data["boxurl"]

            # Network stuff
            node.vm.network :private_network, :auto_network => true
            data["ports"].to_a.each do |port|
                node.vm.network :forwarded_port,
                host:  port["host"],
                guest: port["guest"]
            end

            case data["ostype"]
            when "windows"
                node.vm.communicator = :winrm
                node.vm.hostname = name
                node.vm.guest = :windows
            else
                node.vm.hostname = name + DOMAIN
                node.vm.provision :hosts
                scripts = data["scripts"]
                scripts.to_a.each do |script|
                    node.vm.provision :shell, :path => "files/" + script
                end
            end

            case data["puppet"]
            when "agent"
                node.vm.provision "puppet_server" do |puppet|
                    puppet.puppet_server = data["puppet_server"]
                end
            when "apply"
                if File.exists?("puppet/Puppetfile")
                    node.r10k.puppet_dir = 'puppet'
                    node.r10k.puppetfile_path = 'puppet/Puppetfile'
                    node.r10k.module_path = 'puppet/environments/vagrant/modules'
                end
                node.vm.provision :puppet do |puppet|
                    if File.directory?("puppet/environments/vagrant")
                        puppet.environment_path = "puppet/environments"
                        puppet.environment = "vagrant"
                        puppet.hiera_config_path = "puppet/hiera.yaml"
                        puppet.options = "--verbose --environment vagrant"
                        puppet.working_directory = "/tmp/vagrant-puppet/environments"
                        puppet.module_path = ["puppet/environments/vagrant/modules", "puppet/modules"]
                    end
                end
            else
                # Do nothing :)
            end
        end
    end
end
