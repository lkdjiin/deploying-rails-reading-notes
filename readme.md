Deploying Rails
=====================================

This is my reading notes for the book
[Deploying Rails](http://pragprog.com/book/cbdepra/deploying-rails).

I've bought this book because I'll soon need to launch a rails app with the
following requirements:

* Rails 4
* Ruby 2
* Postgresql

And if I'm lucky, I'll have to launch many little rails apps (as many as
possible in a single server) with:

* Rails 4 (but 3.2 is good enough)
* Ruby 2 (but 1.9.3 is good enough)
* Sqlite3

Chapter 2
===========================

2.1 Installing VirtualBox and Vagrant
-------------------------------------

What I try to install here:

* VirtualBox 4.2.14
* Vagrant 1.2.2
* Ubuntu precise32
* Ruby 1.9.3

VirtualBox 4.2.14 is buggy (at least when used with Vagrant in Debian)
and can't boot boxes. I have to downgrade VirtualBox to version 4.2.10. See
[Vagrant issue](https://github.com/mitchellh/vagrant/issues/1847).

After Ruby install I must do:

    sudo ln -s /usr/local/bin/ruby /opt/vagrant_ruby/bin/ruby

###Information

`apt-get install -y`  
`-y` says yes to all questions.

###To try

Instead of removing system Ruby and compiling Ruby 1.9.3, why not simply try to:

    sudo apt-get install ruby1.9.3

With this method, /opt/vagrant_ruby/bin/ruby still point to version 1.8.7.

###To do

A box with:

* Ruby 2.0
* vim + .vimrc, etc
* .bashrc, alias
* git shortcuts and config

**Edit** (2013-06-24) I think a «base box» with vim, bashrc, git config, etc
but without any other changes (i.e no specific ruby version) will be a good
started point.

    sudo apt-get install vim -y

Copy/paste dotfiles in the project directory (vagrant root) then

    cp /vagrant/.vimrc ~/.vimrc
    cp -r /vagrant/.vim .

And remove dotfiles from project directory. Do similar thing for
`.gitconfig`, `.gemrc` and `.inputrc`. Then package the box:

    vagrant package
    vagrant box remove precise32_with_ruby193 virtualbox
    vagrant box add precise32_with_ruby193 package.box



2.2 Configuring Networks and Multiple Virtual Machines
------------------------------------------------------

###Vagrant settings

Vagrant API is really, really different than that given in the book.

_That sound
very strange to me because the book said it use Vagrant 1.0.2. I'm using Vagrant
1.2.2, so the API should not be broken. Is it Vagrant's fault or book's fault?_

**EDIT** (2013-06-23): It's the Vagrant's behaviour:
[doc](http://docs.vagrantup.com/v2/vagrantfile/version.html). It's like a
versioning system "a la Java", I really hate this thing. (I mean the versioning
system, not Vagrant...)

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
-------------------------------------------

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
------------------------

###My personnal complete Vagrantfile

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


Chapter 3
==============================

3.2 Setting Up Puppet
---------------------

**puppetrails/puppetvm/Vagrantfile**

    Vagrant.configure("2") do |config|
      config.vm.box = "precise32_with_ruby193"
      config.vm.hostname = "app"
      config.vm.network :private_network, ip: "33.33.13.37"
      config.vm.synced_folder "../massiveapp_ops", "/etc/puppet"
      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "512"]
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
      end
    end

To install Puppet, I'll stick close to the book by installing the Puppet gem
version 2.7.22. There is a 3.2.2 available, but I'll look it later.

###To do

Find and install vim syntax for puppet.

**Edit** (2013-06-24) Installed from
[here](https://github.com/puppetlabs/puppet-syntax-vim)


3.3 Installing Apache with Puppet
---------------------------------

Git command lines in the book should have the `-a` switch.

**puppetrails/puppetvm_with_port_80_forwarded/Vagrantfile**

    Vagrant.configure("2") do |config|
      config.vm.box = "precise32_with_ruby193"
      config.vm.hostname = "app"
      config.vm.network :forwarded_port, guest: 80, host: 4567,
                        auto_correct: true
      config.vm.network :private_network, ip: "33.33.13.37"
      config.vm.synced_folder "../massiveapp_ops", "/etc/puppet"
      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "512"]
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
      end
    end

3.4 Configuring MySQL with Puppet
---------------------------------

###Information

`mkdir -p`  
`-p` or `--parents` create parent folders if needed.

###To do

Puppet: package > ensure > present or installed, what is the difference?

I really need to install my git config in the server.

3.5 Creating the MassiveApp Rails Directory Tree
------------------------------------------------

###To do

Review unix file mode (755, 600, etc)

3.6 Writing a Passenger Module
------------------------------

The book says to install passenger 3.0.11. I'll do that for the exercise.
But I'll need passenger 4.x to run Rails 4/Ruby 2, which is my goal.

My first error with Puppet:

    err: /Stage[main]/Passenger/Exec[/usr/local/bin/ 
    passenger-install-apache2-module --auto]/returns: change from notrun
    to 0 failed: /usr/local/bin/passenger-install-apache2-module --auto 
    returned 1 instead of one of [0] at 
    /etc/puppet/modules/passenger/manifests/init.pp:18

No more errors in the console output (is there a Puppet switch for that?).
So I ran:

    /usr/local/bin/passenger-install-apache2-module --auto

And that's tell me I forgot to install `libcurl4-openssl-dev`, so I:

    sudo apt-get install libcurl4-openssl-dev -y


3.8 Updating the Base Box
-------------------------

Instead of

    vagrant box remove precise32_with_ruby193

it is

    vagrant box remove precise32_with_ruby193 virtualbox


Chapter 4
=========================

4.1 Setting Up Capistrano
-------------------------

Book says Capistrano 2.11.2, I'll give a try to version 2.15.4.

4.4 Pushing a Release
---------------------

Capistrano deploy works well. But when `localhost:4567` I see only the
default Apache page. Frustrating...

The `massiveapp_ops/modules/apache2/files/massiveapp.conf` is wrong.
A working version is:

    <VirtualHost *:80>
      ServerName localhost
      DocumentRoot "/var/massiveapp/current/public/"
      CustomLog /var/log/apache2/massiveapp-access_log combined
      ErrorLog /var/log/apache2/massiveapp-error_log
    </VirtualHost>

Then `puppet apply` again and it's work.

Chapter 6
========================

6.1 A MassiveApp to Monitor
---------------------------

    Vagrant.configure("2") do |config|
      config.vm.define :app do |app|
        app.vm.box = "precise32_with_ruby193"
        app.vm.network :forwarded_port, guest: 80, host: 4567, auto_correct: true
        app.vm.hostname = "app"
        app.vm.network :private_network, ip: "33.33.13.37"
        app.vm.synced_folder "../massiveapp_ops", "/etc/puppet"
        app.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "512"]
          vb.customize ["modifyvm", :id, "--cpuexecutioncap", "35"]
        end
      end

      config.vm.define :nagios do |nagios|
        nagios.vm.box = "precise32_with_ruby193"
        nagios.vm.network :forwarded_port, guest: 80, host: 4568, auto_correct: true
        nagios.vm.network :private_network, ip: "33.33.13.38"
        nagios.vm.hostname = "nagios"
        nagios.vm.synced_folder "../massiveapp_ops", "/etc/puppet"
        nagios.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "512"]
          vb.customize ["modifyvm", :id, "--cpuexecutioncap", "35"]
        end
      end
    end

For the rest of this chapter, I'm getting so many errors that I give up
after a fight of 2 hours.


Chapter 7
======================

After bad experience of the previous chapter, I decide to skip this one.


Chapter 8
=====================

A very important chapter, but not for today.


Chapter 9
=====================

9.1 Installing RVM
------------------

Replace

    sudo apt-get install build-essential zlib1g-dev libssl-dev libreadline-dev \
      git-core curl libyaml-dev libcurl4-dev libsqlite3-dev -y

by

    sudo apt-get install build-essential zlib1g-dev libssl-dev libreadline-dev \
      git-core curl libyaml-dev libcurl4-openssl-dev libsqlite3-dev -y

Installing Rvm on Ubuntu precise was really easy. I remember how it was
tedious to install it years ago on my Debian box.

###To do

I'd definitely look at this later:

    You are using '.rvmrc', it requires trusting, it is slower and it is not compatible with other ruby managers,
    you can switch to '.ruby-version' using 'rvm rvmrc to [.]ruby-version'
    or ignore this warnings with 'rvm rvmrc warning ignore /home/vagrant/app193/.rvmrc',
    '.rvmrc' will continue to be the default project file in RVM 1 and RVM 2,
    to ignore the warning for all files run 'rvm rvmrc warning ignore all.rvmrcs'.

9.2 Serving Applications with Passenger Standalone
--------------------------------------------------

I see a promise in this title: no needs for Apache. So I hope far
less config struggle and headache. So cool. But why not start the book
with that?

    Vagrant.configure("2") do |config|
      config.vm.box = "precise32"
      config.vm.network :forwarded_port, guest: 3000, host: 4000, auto_correct: true
      config.vm.network :forwarded_port, guest: 3001, host: 4001, auto_correct: true

      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "512"]
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
      end
    end

###To try

Quoted from the book:

_«The default Rails environment is development, but we could easily started
the app in production mode with Passenger’s `--environment` flag.»_

9.3 Using Systemwide RVM
------------------------

Interesting but not usefull for me, at least now.

9.4 Watching Passenger Standalone with Monit
--------------------------------------------

Not working... Again... One more hour spent for what?

Anyway I should check Monit from its website. This could be a perfect thing
for my needs.


Usefull Documentation
=====================

* [ssh](http://support.suso.com/supki/SSH_Tutorial_for_Linux)
* [Vagrant](http://docs.vagrantup.com/v2/)
* [VirtualBox](https://www.virtualbox.org/wiki/Documentation)
* [Passenger](https://www.phusionpassenger.com/support#documentation)
* [PuppetForge](http://forge.puppetlabs.com/)
* [Capistrano](https://github.com/capistrano/capistrano/wiki)
* [Passenger Standalone](http://www.modrails.com/documentation/Users%20guide%20Standalone.html)
* [Monit](http://mmonit.com/monit/documentation/)
