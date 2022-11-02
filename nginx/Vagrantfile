VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false
  config.vm.provider :virtualbox do |v|
    v.memory = 512
    v.linked_clone = true
  end

  # server 1.
  config.vm.define "master" do |inventory|
    inventory.vm.hostname = "postgres-master"
    inventory.vm.box = "ubuntu/trusty64"
    inventory.vm.network :private_network, ip: "192.168.28.71"
  end

  # server 2.
  config.vm.define "replica" do |inventory|
    inventory.vm.hostname = "replica-2"
    inventory.vm.box = "ubuntu/trusty64"
    inventory.vm.network :private_network, ip: "192.168.28.72"
  end

 # server 3.
  config.vm.define "replica-2" do |inventory|
    inventory.vm.hostname = "replica-2"
    inventory.vm.box = "ubuntu/trusty64"
    inventory.vm.network :private_network, ip: "192.168.28.73"
  end
end

