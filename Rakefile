
client = ENV['client'] || "localhost"
hostname = ENV['hostname'] || "app"
fqd = ENV['fqd'] || "#{hostname}.local"
port = ENV['port'] || 2222
user = ENV['user'] || "vagrant"
private_key = ENV['key'] || "/home/jpratt/.vagrant.d/insecure_private_key"

SSH = "ssh -i #{private_key} -p #{port} -o StrictHostKeyChecking=no -l #{user} #{client}"
REPO = 'https://github.com/blunxy/devbox-rake'

task :default do
  Rake::Task["set_host"].execute
  Rake::Task["install"].invoke("git")
  Rake::Task["set_up_deb_repo"].execute
  Rake::Task["turn_off_auth"].execute
  Rake::Task["unsigned_install"].invoke("my-emacs-24.4")
  Rake::Task["turn_on_auth"].execute
end

desc "Set hostname on ENV['CLIENT'] to ENV['HOSTNAME']"
task :set_host  do
  rename_hostname = "sudo sed -i \"s/precise64/#{hostname}/\" /etc/hostname"
  ssh_command(rename_hostname)

  rename_hosts = "sudo sed -i \"s/precise64/#{fqd} #{hostname}/\" /etc/hosts"
  ssh_command(rename_hosts)

  restart_hostname_service = "sudo service hostname restart"
  ssh_command(restart_hostname_service)
end

task :install, [:name] do |t, args|
  ssh_install args.name
end

task :unsigned_install, [:name] do |t, args|
  ssh_unsigned_install args.name
end



desc "Install Git"
task :install_git do
  ssh_install "git"
end


desc "Add my custom S3 deb repo to sources"
task :set_up_deb_repo do

  ssh_command "echo 'deb https://s3.amazonaws.com/mydebs stable main' | sudo tee -a /etc/apt/sources.list"
  ssh_command "sudo apt-get update"
end

desc "Turn off signed deb checking for my custom debs"
task :turn_off_auth do
  ssh_command "sudo touch /etc/apt/apt.conf.d/99auth"
  ssh_command "echo \"APT::Get::AllowUnauthenticated yes;\" | sudo tee -a /etc/apt/apt.conf.d/99auth"
end

desc "Turn it back on"
task :turn_on_auth do
  ssh_command "sudo rm -f /etc/apt/apt.conf.d/99auth"
end



def ssh_command(cmd)
  sh "#{SSH} '#{cmd}'"
end


def ssh_install(package)
  sh "#{SSH} 'sudo apt-get -y install #{package}'"
end

def ssh_unsigned_install(package)
  sh "#{SSH} 'sudo apt-get -y --force-yes install #{package}'"
end
