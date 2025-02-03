# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
Vagrant.configure("2") do |config|
   config.vm.define "belousovZfs" do |belousovZfs|
    belousovZfs.vm.box = "bento/ubuntu-24.04"      
    belousovZfs.vm.host_name = "belousovZfs"
    belousovZfs.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = "2"
      end

      (1..8).each do |i|
        belousovZfs.vm.disk :disk, size: "1GB", name: "disk-#{i}"
      end
      
      belousovZfs.vm.provision "shell", inline: <<-SHELL
        apt-get updatezfs
        apt install zfsutils-linux -y
      SHELL

   end
end