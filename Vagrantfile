require 'yaml'
kiab_config = YAML::load(File.read("#{File.dirname(__FILE__)}/kiab.config.yaml"))

# Plugins
if ARGV[0] != 'plugin'

    # Define the plugins in an array format
    required_plugins = [
      'vagrant-vbguest', 'vagrant-hostmanager', 
      'vagrant-disksize', 'vagrant-scp'
    ]         
    plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
    if not plugins_to_install.empty?
  
      puts "Installing plugins: #{plugins_to_install.join(' ')}"
      if system "vagrant plugin install #{plugins_to_install.join(' ')}"
        exec "vagrant #{ARGV.join(' ')}"
      else
        abort "Installation of one or more plugins has failed. Aborting."
      end
  
    end
end

playbook_file = 'ansible/buildit-playbook.yml'

Vagrant.configure("2") do |config|
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true
    
    config.vm.box = "geerlingguy/centos7"
    config.vm.hostname = "kiab"
    config.vm.network "private_network", ip: kiab_config.fetch("vm_ip_address", "192.168.100.100")

    config.hostmanager.aliases = %w(kiab.local kiab.localhost kiab)

    config.vm.provider "virtualbox" do |vb|
      vb.cpus = kiab_config.fetch("vm_cpus", 4).to_i
      vb.memory = kiab_config.fetch("vm_memory", 4096).to_i
    end

     # Provision hostnmanager
     config.vm.provision :hostmanager

     # Install with ansible 
     config.vm.provision "ansible_local" do |ansible|
       ansible.playbook = playbook_file
       ansible.verbose = true
       ansible.galaxy_role_file = "ansible/requirements.yml"
     end

    config.trigger.after :up, :provision do |trigger|
        trigger.info = "Copying kubeconfig from VM to local"
        trigger.run = {inline: "bash -c 'vagrant scp :~/.kube/config-kiab ~/.kube/config-kiab'"}
    end 

    if kiab_config.fetch("overwrite_local_kubeconfig", false) 
      config.trigger.after :up, :provision, :reload do |trigger|
        trigger.info = "Overwriting local kubeconfig"
        trigger.run = {inline: "bash -c 'cd ~/.kube && cp config-kiab config'"}
      end 
    end 

end
