# -*- mode: ruby -*-
# vi: set ft=ruby :

# use virtualbox on macos:
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "fedora/33-cloud-base"
  config.vm.box_check_update = false

  config.ssh.forward_agent = true

  config.vm.provider :virtualbox do |virt|
    virt.memory = 4096
    virt.cpus = 2
    # Vagrant configures Virtualbox to port-forward these ports to
    # 127.0.0.1 on the Mac. For example:
    #   http://127.0.0.1:9090 is Prometheus
    #   http://127.0.0.1:3000 is Grafana
    config.vm.network "forwarded_port", guest: 9090, host: 9090
    config.vm.network "forwarded_port", guest: 3000, host: 3000
  end

  config.vm.provider :libvirt do |virt|
    virt.memory = 4096
    virt.cpus = 2
    virt.qemu_use_session = false
    config.vm.network "private_network", ip: "192.168.121.100"
  end

  config.vm.define :grafana do |node|
    node.vm.hostname = 'grafana'
  end

end
