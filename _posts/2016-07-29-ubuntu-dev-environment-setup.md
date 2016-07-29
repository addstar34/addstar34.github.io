---
layout: post
title: Ubuntu Dev Environment Setup
author: Adam D
---

Pre-built development environments are common these days via either a cloud service or sharing of a pre-built vagrant environment, but its always good to know what's under the hood.

I've been using a mix of these depending on where I've been coding and for a long time have been meaning to build my own.

List of packages/software that will be needed
See below for install instructions for these packages
Build-essential # should already be installed
Wget # should already be installed
Openssh-server
Git

Libssl-dev
Libreadline-dev
libffi-dev
Zlig1g-dev
autoconf
bison
libyaml-dev
Libncurses5-dev
libgdbm-dev

Rbenv
Bundler
Postgres
Nodejs
Npm


Update apt-get
This ensures when install packages they’re the latest version
sudo apt-get update

SSH Setup
sudo apt-get install openssh-server
Check service is running
sudo service ssh status
Edit ssh config if required
sudo gedit /etc/ssh/sshd_config
Restart ssh service if needed
sudo service ssh restart
Forward ports for the VM
Protocol: TCP, Host IP: 127.0.0.1, Host Port: 2222, Guest IP: leave blank, Guest Port: 22

Shared folder from host
This requires VBox Guest Additions. Load the VM and from the VM window menu click devices > Insert Guest Additions CD.
Then run
sudo /media/[yourProfileName]/[guestAdditionsCDName]/VBoxLinuxAdditions.run
Create a shared folder from VirtualBox VM > Settings > Shared Folders. Give it a name and select auto mount. It will auto mount to /media/sf_[theShareName].
You might not be able to access this so run:
sudo adduser yourUserName vboxsf
Then reboot

Git
sudo apt-get install git

Required dev packages
These packages are needed before installing ruby.
sudo apt-get install -y autoconf bison libssl-dev libreadline-dev libffi-dev zlib1g-dev libyaml-dev libncurses5-dev libgdbm-dev

Libgdbm3
Libreadline6-dev


Manage Ruby versions with Rbenv
Use the following install instructions incase they get updated. Don’t forget to run the shell specific command.
https://github.com/rbenv/rbenv#installation
Next install ruby-build as a plugin to rbenv to get more power from rbenv:
https://github.com/rbenv/ruby-build#readme
Check list of ruby version
rbenv install -l
Install particular version
rbenv install 2.3.1
Set global ruby version (leave version off to view the current setting)
rbenv global 2.3.1
Set local ruby version for the directory you’re in (leave version off to view the current setting)
rbenv local 2.2.4
List installed versions
rbenv versions

Manage gems with bundler
gem install bundler

Install Postgres
Sudo apt-get install postgresql-9.5 postgresql-contrib-9.5 postgresql-server-dev-9.5
After installed check installation:
sudo su - postgres
psql
\conninfo
create a new ‘user’ or as postgres calls it a role
CREATE ROLE pgdba with CREATEDB LOGIN PASSWORD ‘password’;
This will create a username called pgdba with the password of password
To exit pgsql type
\q
You will now be logged in as postgres so switch back to your account
su - username
Edit pg_hba.conf to use password login instead of peer (this is for rails)
First backup pg_hba.conf
sudo cp -n /etc/postgresql/9.5/main/pg_hba.conf /etc/postgresql/9.5/main/pg_hba.backup
Then edit this file
sudo gedit /etc/postgresql/9.5/main/pg_hba.conf
And replace
# TYPE DATABASE USER ADDRESS METHOD
Local all all peer
to
Local all all md5

Install Nodejs
Check instructions at https://nodejs.org/ but at the time of writing these were:
curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
sudo apt-get install -y nodejs
Note: look into using nvm (node version manager)

Manage rails versions via bundler
https://relativkreativ.at/articles/managing-multiple-rails-versions
To check rails versions:
gem search rails | grep “^rails “
Create a rails project and specify version:
cd /projects
mkdir rails-app-1
echo “source ‘https://rubygems.org’” > Gemfile
echo “gem ‘rails’, ‘4.1.0’” >> Gemfile
bundle install
Check the correction version has been installed using:
bundle exec rails -v
Now create your application, let Rails create a new Gemfile or rather overwrite the existing one (--force) and install of install the bundle (--skip-bundle) update it manually:
bundle exec rails new . --force --skip-bundle --database=postgresql
bundle update
Note: older version weren’t always rails new if the command fails.
Update database.yml to use the postgres user pgdba and password set in the postgres section.
Create the databases
Rake db:create:all or rails db:create:all
Before starting the rails server forward a port in VM:
Protocol: TCP, Host IP: leave blank, Host Port: 3030, Guest IP: leave blank, Guest Port: 3000
Start rails server like so
rails s -b 0.0.0.0

Do the above steps again to create another project with a different version. Then you can check versions using:
gem list --local

Disable Ubuntu GUI
From within the Ubuntu Gui
First make a backup
sudo cp -n /etc/default/grub /etc/default/grub.orig
Then if you need to revert run:
sudo mv /etc/default/grub.orig /etc/default/grub && sudo update-grub
Edit bootloader
sudo gedit /etc/default/grub
Edit this line:
GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash”
To this:
GRUB_CMDLINE_LINUX_DEFAULT=”text”
Tell systemd to not load the graphical login manager
sudo systemctl enable multi-user.target --force
sudo systemctl set-default multi-user.target
To start the GUI from this state from the VM terminal run
sudo /etc/init.d/lightdm start

From now on when starting the vm start it in headless mode.
