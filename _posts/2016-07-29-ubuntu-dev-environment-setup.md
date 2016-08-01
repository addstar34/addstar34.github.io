---
layout: post
title: Ubuntu Dev Environment Setup
author: Adam D
---

Pre-built development environments are common these days via either a cloud service or sharing of a pre-built vagrant environment, but its always good to know what's under the hood.

###### **This is my no fluff guide to building a Ubuntu Dev Environment in VirtualBox.**

This guide will get you to the stage where you can start up a rails project but it does include Nodejs and NPM. It also includes:

SSH setup, shared folders between your host machine and Ubuntu (so can always access your code without having to power on the VirtualBox), managing and installing Ruby versions with Rbenv, installing Postgresql, managing Rails versions and gems with bundler, installing nodejs and finally disabling the Ubuntu GUI so you can start the VirtualBox in headless mode and just use a terminal.


## List of packages/software that will be needed
- Ubuntu 16.04  
- VirtualBox 5+  
- Build-essential *# should already be installed*  
- Wget *# should already be installed*  
- Openssh-server  
- Git  
- libssl-dev  
- libreadline-dev  
- libffi-dev  
- zlig1g-dev  
- autoconf  
- bison  
- libyaml-dev  
- libncurses5-dev  
- libgdbm-dev  
- Rbenv  
- Bundler  
- Postgresql  
- Nodejs  
- Npm  
- Heroku Toolbelt

## Update apt-get
This ensures when installing packages they’re the latest version  
`sudo apt-get update`

## SSH Setup
Install OpenSSH Server  
`sudo apt-get install openssh-server`  
Check service is running  
`sudo service ssh status`  
Edit ssh config if required  
`sudo gedit /etc/ssh/sshd_config`  
Restart ssh service if needed  
`sudo service ssh restart`  
Forward ports in VirtualBox under Settings > Network > Port Forwarding  
`Protocol: TCP, Host IP: 127.0.0.1, Host Port: 2222, Guest IP: leave blank, Guest Port: 22`

## Shared folder from host
This requires VBox Guest Additions. Load the VM and from the VM window menu click devices > Insert Guest Additions CD. Then run  
`sudo /media/[yourProfileName]/[guestAdditionsCDName]/VBoxLinuxAdditions.run`  
Create a shared folder from VirtualBox VM > Settings > Shared Folders. Give it a name and select auto mount. It will auto mount to /media/sf_[ShareName].  
To access this your user needs vboxsf rights so run  
`sudo adduser yourUserName vboxsf`  
Then reboot

Remember to create your projects in /media/sf_[ShareName] so you can always access them without having to power up the virtual box.

## VirtualBox Shared Folder enable Symlinks
You might run into some errors about 'erofs read-only file system symlink' when using shared folders and downloading some npm packages. For example I ran into this issue when downloading react to a shared project folder.  
To fix this you need to run the following on your shared folder  
`VBboxManage setextradata "[VM Name]" VBoxInternal2/SharedFoldersEnableSymlinksCreate/[sharename] 1`  
note this is 1 line to execute, replace [VM Name] and [sharename]. For example on windows I opened command and ran  
'"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" setextradata "ubuntu1604" VBoxInternal2/SharedFoldersEnableSymlinksCreate/dev 1'

## Git
Install Git  
`sudo apt-get install git`

## Required dev packages
These packages are needed before installing ruby  
`sudo apt-get install -y autoconf bison libssl-dev libreadline-dev libffi-dev zlib1g-dev libyaml-dev libncurses5-dev libgdbm-dev`

## Manage Ruby versions with Rbenv
Use the following install instructions in case they get updated. Don’t forget to run the shell specific command.  
https://github.com/rbenv/rbenv#installation  
Next install ruby-build as a plugin to rbenv to get more power from rbenv  
https://github.com/rbenv/ruby-build#readme  
Check list of ruby version
`rbenv install -l`  
Install particular version  
`rbenv install 2.3.1`  
Set global ruby version (leave version off to view the current setting)  
`rbenv global 2.3.1`  
Set local ruby version for the directory you’re in (leave version off to view the current setting)  
`rbenv local 2.2.4`  
List installed versions  
`rbenv versions`

## Manage gems with bundler
Install Bundler  
`gem install bundler`

## PostgreSQL
Install Postresql  
`Sudo apt-get install postgresql-9.5 postgresql-contrib-9.5 postgresql-server-dev-9.5`  
After installed check installation  
`sudo su - postgres`  
`psql`  
`\conninfo`  
create a new user or as postgres calls it a role  
`CREATE ROLE pgdba with CREATEDB LOGIN PASSWORD ‘password’;`  
This will create a user name called pgdba with the password of password  
To exit pgsql type  
`\q`  
You will now be logged in as postgres so switch back to your account  
`su - username`
Edit pg_hba.conf to use password login instead of peer (this is for rails)  
First backup pg_hba.conf  
`sudo cp -n /etc/postgresql/9.5/main/pg_hba.conf /etc/postgresql/9.5/main/pg_hba.backup`  
Then edit this file  
`sudo gedit /etc/postgresql/9.5/main/pg_hba.conf`  
And the following line  
`## TYPE DATABASE USER ADDRESS METHOD`  
`Local all all peer`  
to  
`Local all all md5`

## Install Nodejs
Check instructions at https://nodejs.org/ but at the time of writing these were  
`curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -`  
`sudo apt-get install -y nodejs`  
Note to self: look into using nvm (node version manager)

## Update NPM and Fix NPM Permissions
Nodejs includes NPM run the following to update to the latest version  
`sudo npm install npm -g`  
Install a package  
`npm install -g jshint`  
If you get an EACCS error there are 3 fixes explained here  
https://docs.npmjs.com/getting-started/fixing-npm-permissions  
I went with option 2 Change npm's default directory  
`mkdir ~/.npm-global`  
Configure npm to use the new directory path  
`npm config set prefix '~/.npm-global'`  
Add the following line to ~/.profile  
`echo 'export PATH=~/.npm-global/bin:$PATH >> ~/.profile'`  
Update system variables  
`source ~/.profile`  
Test downloading a package globally without using sudo  
`npm install -g jshint`

## Manage rails versions via bundler
Create a rails project and specify version  
`cd /projects`  
`mkdir rails-app-1`  
`echo "source 'https://rubygems.org'" > Gemfile`  
`echo "gem 'rails', '5.0'" >> Gemfile`  
`bundle install`  
Check the correction version has been installed using  
`bundle exec rails -v`  
Now create your application. Use --force to overwrite the existing Gemfile and --skip-bundle so you can manually run it later and --database=postgresql to use postgres.  
`bundle exec rails new . --force --skip-bundle --database=postgresql`  
`bundle update`  
Update database.yml to use the postgres user pgdba and password set in the postgres section.  
Create the databases  
`Rake db:create:all` or `rails db:create:all`  
Before starting the rails server forward a port in VirtualBox  
`Protocol: TCP, Host IP: leave blank, Host Port: 3030, Guest IP: leave blank, Guest Port: 3000`  
Start rails server setting the IP address  
`rails s -b 0.0.0.0`

You can repeat these steps to create another project with a different rails version.  

## Heroku Toolbelt
Check the latest instructions here  
https://toolbelt.heroku.com  
At the time of writing this was  
`wget -O- https://toolbelt.heroku.com/install-ubuntu.sh | sh`  
Install heroku accounts for managing multiple accounts i.e. personal, work and/or client  
`heroku plugins:install heroku-accounts`  

## Disable Ubuntu GUI
From within the Ubuntu Gui  
First make a backup  
`sudo cp -n /etc/default/grub /etc/default/grub.orig`  
Then if you need to revert run  
`sudo mv /etc/default/grub.orig /etc/default/grub && sudo update-grub`  
Edit bootloader  
`sudo gedit /etc/default/grub`  
Edit this line  
`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`  
To this  
`GRUB_CMDLINE_LINUX_DEFAULT="text"`  
Tell systemd to not load the graphical login manager  
`sudo systemctl enable multi-user.target --force`  
`sudo systemctl set-default multi-user.target`  
To start the GUI from this state from the VM terminal run  
`sudo /etc/init.d/lightdm start`

From now on when starting the vm start it in headless mode and access via terminal.

Now doesn't that feel good working with a nice clean setup :)
