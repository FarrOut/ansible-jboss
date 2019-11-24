Vagrant.require_version ">= 1.8.0"

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"

  config.vm.provision "ansible" do |ansible|    
    ansible.verbose = true
    ansible.playbook = "provisioning/play-jboss.yml"
    ansible.limit = 'all,localhost'
    ansible.host_vars = {
      "localhost" => {"ansible_become_pass" => 'vagrant'}
    }
  end
end
