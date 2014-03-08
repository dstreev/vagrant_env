# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Configure the domain you'd like to use for your vm's.
domain = "hwx.test"
bridge_interface = "en5: Thunderbolt Ethernet"

# Identify the vm's by name, ipaddress, cpu's, memory and the roles and additional recipes.
boxes = [
    {:name => "u1-denali", :ipaddress => "10.0.6.41", :netmask => "255.255.0.0", :gateway => "10.0.0.1", :cpus => 2, :memory => 3072, :roles => ["base"], :recipes => []},
    {:name => "u2-denali", :ipaddress => "10.0.6.42", :netmask => "255.255.0.0", :gateway => "10.0.0.1", :cpus => 2, :memory => 3072, :roles => ["base"], :recipes => []},
    {:name => "u3-denali", :ipaddress => "10.0.6.43", :netmask => "255.255.0.0", :gateway => "10.0.0.1", :cpus => 2, :memory => 3072, :roles => ["base"], :recipes => []},
    {:name => "u4-denali", :ipaddress => "10.0.6.44", :netmask => "255.255.0.0", :gateway => "10.0.0.1", :cpus => 2, :memory => 3072, :roles => ["base"], :recipes => []}
]

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Change to match your copy of the vagrant private ssh-key used by the box.  
  config.ssh.private_key_path = "/Users/%s/.ssh/vagrant" % ENV['USER'].to_s

  config.ssh.username = "vagrant"

# CURRENT HAVING ISSUE WITH /etc/hosts adjustments.  It's setting everything to 127.0.0.1, so we're managing those manually for now.  
#  config.vm.provision :hostmanager
#  config.hostmanager.enabled = true
#  config.hostmanager.manage_host = true
#  config.hostmanager.ignore_private_ip = true
#  config.hostmanager.include_offline = true

  boxes.each do |opts|
    config.vm.define opts[:name] do |vmconfig|
      # CentOS 6.4 x86_64 Minimal (VirtualBox Guest Additions 4.2.12, Chef 11.4.4, Puppet 3.1.1)
      
      # To use the box, download it from the following URL:
      # http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130427.box
      # Save it locally and add it to the vagrant application with:
      # vagrant box add "CentOS_64_x64" <location_of_the_local_box_file> --provider virtualbox
      vmconfig.vm.box = "CentOS_64_x64"

      vmconfig.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--cpus", opts[:cpus]]
        vb.customize ["modifyvm", :id, "--memory", opts[:memory]]
      end

	  # Using the above "domain" value to set the hostname.
      vmconfig.vm.hostname = "%s" % opts[:name]+"."+domain.to_s
      #vmconfig.vm.network :private_network, ip: opts[:ipaddress], netmask: opts[:netmask]  
      vmconfig.vm.network :public_network, :bridge => :bridge_interface, ip: opts[:ipaddress], netmask: opts[:netmask]  
	  
	  # Add host aliases to /etc/hosts
      vmconfig.hostmanager.aliases = opts[:name]

      vmconfig.vm.provision :chef_solo do |chef|
        chef.cookbooks_path = ["cookbooks", "site-cookbooks"]
        chef.data_bags_path = "databags"
        chef.roles_path = "roles"
		
		
        opts[:roles].each do |role|
          chef.add_role(role)
        end

        opts[:recipes].each do |recipe|
          chef.add_recipe(recipe)
        end
        
        # Pass the domain to the recipes.
        chef.json = {
    		"default_attributes" => {
        		"hdp-prep" => {
            		"domain" => {
            			"name" => :domain
            		}
        		}
    		}
  		}
      end
    end
  end
end
