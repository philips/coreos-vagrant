# -*- mode: ruby -*-
# # vi: set ft=ruby :

require_relative 'override-plugin.rb'

NUM_INSTANCES = 1

CLOUD_CONFIG_PATH = "./user-data"

$script = <<SCRIPT
mkdir -p /var/lib/coreos
mv /tmp/user-data /var/lib/coreos/user-data
cat << "EOF" > /run/systemd/system/vagrant-coreos-cloudinit.service
[Unit]
Description=Run cloudinit using Vagrant user-data
[Service]
Type=oneshot
ExecStart=/usr/bin/coreos-cloudinit --from-file /var/lib/coreos/user-data
[Install]
WantedBy=multi-user.target
EOF

mkdir -p /etc/systemd/system
cp -Ra /run/systemd/system/vagrant-coreos-cloudinit.service /etc/systemd/system/vagrant-coreos-cloudinit.service
systemctl enable /etc/systemd/system/vagrant-coreos-cloudinit.service

systemctl daemon-reload
systemctl start vagrant-coreos-cloudinit.service
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "coreos-alpha"
  config.vm.box_url = "http://storage.core-os.net/coreos/amd64-usr/alpha/coreos_production_vagrant.box"

  # Fix docker not being able to resolve private registry in VirtualBox
  config.vm.provider :virtualbox do |vb, override|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://storage.core-os.net/coreos/amd64-generic/dev-channel/coreos_production_vagrant_vmware_fusion.box"
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  (1..NUM_INSTANCES).each do |i|
    config.vm.define vm_name = "core-%02d" % i do |config|
      config.vm.hostname = vm_name

      ip = "172.12.8.#{i+100}"
      config.vm.network :private_network, ip: ip

      # Uncomment below to enable NFS for sharing the host machine into the coreos-vagrant VM.
      #config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']

      if File.exist?(CLOUD_CONFIG_PATH)
        config.vm.provision :file, :source => "#{CLOUD_CONFIG_PATH}", :destination => "/tmp/user-data"
        config.vm.provision :shell, :inline => $script, :privileged => true
      end

    end
  end
end
