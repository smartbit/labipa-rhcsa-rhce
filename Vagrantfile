# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
  config.ssh.insert_key = false # for vagrant 1.8.5 see https://github.com/mitchellh/vagrant/issues/7610#issuecomment-234609660
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vbguest.auto_update = false
  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
  end
  config.vm.network "private_network", ip: "192.168.4.200"


#  config.vm.provision "shell", inline: "export LANG=en_US.utf8 ; echo 'export LANG=en_US.utf8\nexport LC_ALL=en_US.utf8' > /etc/profile.d/custom.sh; chmod +x /etc/profile.d/custom.sh; yum install -y python2 > /dev/null"
  config.vm.provision "shell", inline: "export LANG=en_US.utf8; localectl set-locale LANG=en_US.utf8; echo 'export LANG=en_US.utf8\nexport LC_ALL=en_US.utf8' > /etc/profile.d/custom.sh; chmod +x /etc/profile.d/custom.sh"
  config.vm.hostname = "labipa"
  config.vm.provision :ansible do |ansible|
      ansible.playbook = "labipa.yml"
      ansible.inventory_path = "labipa"
      ansible.verbose = true
      ansible.limit = "all"
  end
end
