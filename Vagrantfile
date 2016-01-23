VAGRANTFILE_API_VERSION = "2"
BOX = "ubuntu/trusty64"

base_dir = File.expand_path(File.dirname(__FILE__))

# Cluster, VM and network settings
NETWORK_SUBNET = "192.168.1"
NETWORK_DOMAIN = "lan"
NUM_DOCKER_ENGINES = 2
NUM_DOCKER_REGISTRIES = 1
DOCKER_ENGINE_IPS_START = 10
DOCKER_REGISTRY_IPS_START = 200
DOCKER_ENGINE_MEMORY = 2048
DOCKER_REGISTRY_MEMORY = 512
DOCKER_ENGINE_CPU = 2
DOCKER_REGISTRY_CPU = 1

ansible_provision = proc do |ansible|
  ansible.playbook = 'site.yml'
  # Note: Can't do ranges like mon[0-2] in groups because
  # these aren't supported by Vagrant, see
  # https://github.com/mitchellh/vagrant/issues/3539
  ansible.groups = {
    'docker_engines'    => (0..NUM_DOCKER_ENGINES - 1).map { |j| "docker#{j}.#{NETWORK_DOMAIN}" },
    'docker_registries' => (0..NUM_DOCKER_REGISTRIES - 1).map { |j| "registry#{j}.#{NETWORK_DOMAIN}" },
    'all:children'      => ["docker_engines", "docker_registries"],
  }

  # In a production deployment, these should be secret
  ansible.extra_vars = {
    cluster_network: "#{NETWORK_SUBNET}.0/24",
  }
  ansible.limit = 'all'
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.ssh.insert_key = false
  config.vm.box = BOX

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :machine
    config.cache.enable :apt
  end
  
  (0..NUM_DOCKER_ENGINES - 1).each do |i|
    config.vm.define "docker#{i}.#{NETWORK_DOMAIN}" do |de|
      de.vm.hostname = "docker#{i}.#{NETWORK_DOMAIN}"
      de.vm.network :private_network, ip: "#{NETWORK_SUBNET}.#{DOCKER_ENGINE_IPS_START + i}"

      # Virtualbox
      de.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', "#{DOCKER_ENGINE_MEMORY}"]
        vb.customize ['modifyvm', :id, '--cpus', "#{DOCKER_ENGINE_CPU}"]
      end

      # VMware
      de.vm.provider :vmware_fusion do |v|
        v.vmx['memsize'] = "#{DOCKER_ENGINE_MEMORY}"
      end

      # Libvirt
      de.vm.provider :libvirt do |lv|
        lv.memory = DOCKER_ENGINE_MEMORY
      end
    end
  end
  
  (0..NUM_DOCKER_REGISTRIES - 1).each do |i|
    config.vm.define "registry#{i}.#{NETWORK_DOMAIN}" do |dr|
      dr.vm.hostname = "registry#{i}.#{NETWORK_DOMAIN}"
      dr.vm.network :private_network, ip: "#{NETWORK_SUBNET}.#{DOCKER_REGISTRY_IPS_START + i}"

      # Virtualbox
      dr.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', "#{DOCKER_REGISTRY_MEMORY}"]
        vb.customize ['modifyvm', :id, '--cpus', "#{DOCKER_REGISTRY_CPU}"]
      end

      # VMware
      dr.vm.provider :vmware_fusion do |v|
        v.vmx['memsize'] = "#{DOCKER_REGISTRY_MEMORY}"
      end

      # Libvirt
      dr.vm.provider :libvirt do |lv|
        lv.memory = DOCKER_REGISTRY_MEMORY
      end
      
      # Run the provisioner after the last machine comes up
      dr.vm.provision 'ansible', &ansible_provision if i == (NUM_DOCKER_REGISTRIES - 1)
    end
  end
end