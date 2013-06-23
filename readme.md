Deploying Rails
=====================================

My reading notes for the book
[Deploying Rails](http://pragprog.com/book/cbdepra/deploying-rails).


2.1 Installing VirtualBox and Vagrant
=====================================

What I try to install:

* VirtualBox 4.2.14
* Vagrant 1.2.2
* Ubuntu precise32
* Ruby 1.9.3

VirtualBox 4.2.14 is buggy (at least when used with Vagrant) and can't boot
boxes. I have to downgrade VirtualBox to version 4.2.10. See
[Vagrant issue](https://github.com/mitchellh/vagrant/issues/1847).

After Ruby install I must do:

    sudo ln -s /usr/local/bin/ruby /opt/vagrant_ruby/bin/ruby


To try
----------------------

Instead of removing system Ruby and compiling Ruby 1.9.3, why not simply try to:

    sudo apt-get install ruby1.9.3
    sudo update-alternatives --config ruby

To do
---------------------

A box with:

* Ruby 2.0
* vim + .vimrc, etc
* .bashrc


2.2 Configuring Networks and Multiple Virtual Machines
======================================================

Vagrant settings
------------------------------------

Vagrant API is really, really different than that given in the book.

_That sound
very strange to me because the book said it use Vagrant 1.0.2. I'm using Vagrant
1.2.2, so the API should not be broken. Is it Vagrant's fault or book's fault?_

    config.vm.provider :virtualbox do |vb|
      vb.name = "UbuntuPrecise32"
      vb.customize ["modifyvm", :id, "--memory", "512"]
      # I find this one very interesting.
      vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
    end
    config.vm.network :private_network, ip: "33.33.13.37"
    config.vm.network :forwarded_port, guest: 80, host: 4567, auto_correct: true

Shared folder is automatic:
[source](http://docs.vagrantup.com/v2/synced-folders/index.html)


2.3 Running Multiple VMs
===========================================

Settings that worked for me:

    Vagrant.configure("2") do |config|
      config.vm.define :app do |app|
        app.vm.box = "precise32_with_ruby193"
        app.vm.network :forwarded_port, guest: 80, host: 4567, auto_correct: true
        app.vm.hostname = "app"
        app.vm.network :private_network, ip: "33.33.13.37"
        app.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "512"]
          vb.customize ["modifyvm", :id, "--cpuexecutioncap", "35"]
        end
      end

      config.vm.define :db do |db|
        db.vm.box = "precise32_with_ruby193"
        db.vm.network :private_network, ip: "33.33.13.38"
        db.vm.hostname = "db"
        db.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "512"]
          vb.customize ["modifyvm", :id, "--cpuexecutioncap", "35"]
        end
      end
    end

Do not specify:

    app.vm.network :forwarded_port, guest: 22, host: 2222, auto_correct: true

This is done automatically, and even could prevent your VMs to boot.

2.6 For Future Reference
========================

My personnal complete Vagrantfile
---------------------------------

    # vi: set ft=ruby :
    Vagrant.configure("2") do |config|
      config.vm.box = "precise32"
      config.vm.network :forwarded_port, guest: 80, host: 4567,
                        auto_correct: true
      config.vm.hostname = "app"
      config.vm.network :private_network, ip: "33.33.13.37"
      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "512"]
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
      end
    end

Usefull Documentation
=====================

* [ssh](http://support.suso.com/supki/SSH_Tutorial_for_Linux)
* [Vagrant](http://docs.vagrantup.com/v2/)
* [VirtualBox](https://www.virtualbox.org/wiki/Documentation)
