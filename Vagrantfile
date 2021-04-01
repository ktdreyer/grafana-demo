# -*- mode: ruby -*-
# vi: set ft=ruby :

# use virtualbox on macos:
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "fedora/33-cloud-base"
  config.vm.box_check_update = false

  config.ssh.forward_agent = true

  config.vm.provider :libvirt do |virt|
    virt.memory = 4096
    virt.cpus = 2
    virt.qemu_use_session = false
    config.vm.network "private_network", ip: "192.168.121.100"
  end

  config.ssh.forward_agent = true

  config.vm.define :grafana do |node|
    node.vm.hostname = 'grafana'
  end

end
