
client = ENV['client'] || "localhost"
hostname = ENV['hostname'] || "app"
fqd = ENV['fqd'] || "#{hostname}.local"
port = ENV['port'] || 2222
user = ENV['user'] || "vagrant"
private_key = ENV['key'] || "/home/jpratt/.vagrant.d/insecure_private_key"

SSH = "ssh -i #{private_key} -p #{port} -o StrictHostKeyChecking=no -l #{user} #{client}"
REPO = 'https://github.com/blunxy/puppet-devmachine.git'

task :default do
  Rake::Task["set_host"].execute
  Rake::Task["install_puppet"].execute
  Rake::Task["install_git"].execute
  Rake::Task["pull_puppet_structure"].execute
  Rake::Task["set_up_papply"].execute
  Rake::Task["set_up_deb_repo"].execute
  Rake::Task["run_papply"].execute
end

desc "Run papply"
task :run_papply do
  ssh_command "papply"
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

desc "Install Puppet"
task :install_puppet do
    ssh_command "wget http://apt.puppetlabs.com/puppetlabs-release-precise.deb"
    ssh_command "sudo dpkg -i puppetlabs-release-precise.deb"
    ssh_command "sudo apt-get update && sudo apt-get -y install puppet"
end

desc "Install Git"
task :install_git do
  ssh_command "sudo apt-get -y install git"
end

desc "Pull down Puppet structure from Github."
task :pull_puppet_structure do
  ssh_command "sudo rm -rf /etc/puppet"
  ssh_command "sudo git clone #{REPO} /etc/puppet"
  ssh_command "sudo rm -f /etc/puppet/Rakefile"
end

desc "Set up papply script."
task :set_up_papply do
  ssh_command "sudo puppet apply --modulepath=/etc/puppet/modules -e \"include convenient_scripts\" -v"
end

desc "Add my custom S3 deb repo to sources"
task :set_up_deb_repo do
  ssh_command "echo 'deb https://s3.amazonaws.com/mydebs stable main' | sudo tee -a /etc/apt/sources.list"
  ssh_command "sudo apt-get update"
end

def ssh_command(cmd)
  sh "#{SSH} '#{cmd}'"
end
