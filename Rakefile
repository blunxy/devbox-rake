#
# First stab at making a provisioning "script". Got here because first started
# down the Puppet road thanks to the Provisioning Rails book...which led me to
# buy the Puppet 3 Cookbook book...which had a recipe that used Rake to bootstrap
# a Puppet standalone install...which made me think that I should just stay with
# Rake when some Puppet annoyances arose.
#
# In any case, this script has the benefit of laying a lot of the steps "bare" -
# you can see how things are done fairly clearly. I've tried to tidy things up
# a bit, though I could spend more time using more Rake features, I'm sure.
#
# One other benefit is that this can be used to get a production server for
# FallCon up an running...with some tweaking, of course.
#
# JP
# 2014-May
#

client = ENV['client'] || "localhost"
hostname = ENV['hostname'] || "app"
fqd = ENV['fqd'] || "#{hostname}.local"
port = ENV['port'] || 2222
user = ENV['user'] || "vagrant"
private_key = ENV['key'] || "/home/jpratt/.vagrant.d/insecure_private_key"

SSH = "ssh -i #{private_key} -p #{port} -o StrictHostKeyChecking=no -l #{user} #{client}"
REPO = 'https://github.com/blunxy/devbox-rake'

task :default do
  run "set_hostname"
  run "install", "tree"
  run "install", "git"
  run "install", "curl"
  run "install", "tmux"
  run "set_up_the_silver_searcher"
  run "set_up_deb_repo"
  run "unsigned_install", "my-emacs-24.4"
  run "remove_old_ruby"
  run "install_rvm"
  run "install_ruby"
  run "gem_install", "bundler"
  run "check_gems"
  run "install_postgres"
  run "create_vagrant_postgres_user"
  run "install_nodejs"
end

task :set_up_the_silver_searcher do
  ssh_command "sudo apt-get update"
  ssh_install "make"
  ssh_install "automake"
  ssh_install "pkg-config"
  ssh_install "libpcre3-dev"
  ssh_install "zlib1g-dev"
  ssh_install "liblzma-dev"
  ssh_command "git clone https://github.com/ggreer/the_silver_searcher.git"
  init_ag_script
  ssh_command "sudo su -c /home/vagrant/ag_install.sh"
end

desc "Set hostname on ENV['CLIENT'] to ENV['HOSTNAME']"
task :set_hostname  do
  rename_hostname = "sudo sed -i \"s/precise64/#{hostname}/\" /etc/hostname"
  ssh_command(rename_hostname)

  rename_hosts = "sudo sed -i \"s/precise64/#{fqd} #{hostname}/\" /etc/hosts"
  ssh_command(rename_hosts)

  restart_hostname_service = "sudo service hostname restart"
  ssh_command(restart_hostname_service)
end

task :install_nodejs do
  ssh_command "sudo apt-get update"
  ssh_command "sudo apt-get install -y  python-software-properties"
  ssh_command "sudo add-apt-repository -y  ppa:chris-lea/node.js"
  ssh_command "sudo apt-get update"
  ssh_command "sudo apt-get install -y python-software-properties python g++ make nodejs"
end

task :install, [:name] do |t, args|
  ssh_install args.name
end

task :gem_install, [:name] do |t, args|
  ssh_gem_install args.name
end

task :check_gems do
  init_gem_script
  ssh_command "sudo su -c /home/vagrant/gem_update.sh vagrant"
end

task :unsigned_install, [:name] do |t, args|
  ssh_unsigned_install args.name
end

desc "In goes RVM"
task :install_rvm do
  ssh_command "\\curl -sSL https://get.rvm.io | bash -s stable"
  ssh_command "source /home/vagrant/.rvm/scripts/rvm"
end




desc "In goes Ruby"
task :install_ruby do
  ssh_command "sudo apt-get -o Dpkg::Options::=\"--force-overwrite\" -y install autoconf"
  ssh_command "/home/vagrant/.rvm/bin/rvm install 2.1.1"
  ssh_command "/home/vagrant/.rvm/bin/rvm install 1.9.3"
  init_rvm_script
  ssh_command "sudo su -c /home/vagrant/default_ruby.sh vagrant"
end

def ssh_gem_install(name, ruby_version="2.1.1")
  contents = "#!/usr/bin/env bash\nsource /home/vagrant/.rvm/scripts/rvm\n/home/vagrant/.rvm/rubies/ruby-#{ruby_version}/bin/gem install #{name} --no-ri --no-rdoc"
  init_script contents, "/home/vagrant/bundle.sh"
  ssh_command "sudo su -c /home/vagrant/bundle.sh vagrant"
  ssh_command "rm -f /home/vagrant/bundle.sh"
end

desc "In goes Postgres"
task "install_postgres" do
  ssh_command "sudo touch /etc/apt/sources.list.d/pgdg.list"
  ssh_command "echo 'deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main' | sudo tee -a /etc/apt/sources.list.d/pgdg.list"
  ssh_command "wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - sudo apt-get update"
  ssh_command "sudo /usr/sbin/update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8"
  ssh_command "sudo apt-get update"
  ssh_install "postgresql-9.3"
  ssh_install "libpq-dev"
  ssh_command "sudo mkdir -p /usr/local/pgsql/data"
  ssh_command "sudo chown postgres:postgres /usr/local/pgsql/data"
  ssh_command "sudo /etc/init.d/postgresql start"
end

task :create_vagrant_postgres_user do
  init_postgres_script
  ssh_command "sudo su -c /var/lib/postgresql/init.sh postgres"
end

desc "Add my custom S3 deb repo to sources"
task :set_up_deb_repo do
  ssh_command "echo 'deb https://s3.amazonaws.com/mydebs stable main' | sudo tee -a /etc/apt/sources.list"
  ssh_command "sudo apt-get update"
end

desc "Remove old ruby"
task :remove_old_ruby do
  oldpacks = ["ruby","ruby1.8","ruby1.9.1","libruby","libruby1.8","libruby1.9.1"]
  oldpacks.each do |pack|
    ssh_purge pack
  end
end
;
def init_script(contents, source, as_user = "vagrant")
  ssh_command "sudo -u #{as_user} touch #{source}"
  ssh_command "sudo -u #{as_user} chmod +x #{source}"
  ssh_command "echo \"#{contents}\" | sudo -u #{as_user} tee -a #{source}"
end


def init_rvm_script
  script_contents = "#!/usr/bin/env bash\nsource /home/vagrant/.rvm/scripts/rvm\nrvm --default use 2.1.1"
  init_script script_contents, "/home/vagrant/default_ruby.sh", "vagrant"
end

def init_ag_script
  script_contents = "#!/usr/bin/env bash\ncd the_silver_searcher && ./build.sh && sudo make install"
  init_script script_contents, "/home/vagrant/ag_install.sh", "vagrant"
end


def init_gem_script
  script_contents = "#!/usr/bin/env bash\nsource /home/vagrant/.rvm/scripts/rvm\nrvm gemset use global\ngem update\necho \"gem: --no-document\" >> ~/.gemrc"
  init_script script_contents, "/home/vagrant/gem_update.sh", "vagrant"
end

def init_postgres_script
  script_contents = "#!/bin/bash\n/usr/lib/postgresql/9.3/bin/initdb -D /usr/local/pgsql/data\ncreateuser vagrant --createdb\nexit"
  init_script script_contents, "/var/lib/postgresql/init.sh", "postgres"
end

def ssh_command(cmd)
  sh "#{SSH} '#{cmd}'"
end

def ssh_install(package)
  sh "#{SSH} 'sudo apt-get -y install #{package}'"
end

def ssh_purge(package)
  sh "#{SSH} 'sudo apt-get -y purge #{package}'"
end

def ssh_unsigned_install(package)
  sh "#{SSH} 'sudo apt-get -y --force-yes install #{package}'"
end

def run(taskname, args=nil)
  if (args.nil?)
    Rake::Task["#{taskname}"].execute
  else
    Rake::Task["#{taskname}"].invoke("#{args}")
    Rake::Task["#{taskname}"].reenable
  end
end
