# -*- mode: ruby -*-
# vi: set ft=ruby :

## Define some app-specific stuff to be used later during provisioning: ##
app_vars = {
  appname: 'MyApplication',
  dbname: 'symfony',
  dbuser: 'vagrant',
  dbpassword: 'vagrant',
  dbtype: 'postgresql' # 'mysql' or 'postgresql'
  box_memory: 2048
}
# ansible_verbosity = 'vvvv'
##########################################################################

#Fix for people with strange locale settings
ENV.keys.each {|k| k.start_with?('LC_') && ENV.delete(k)}

def host_box_is_unixy?
  (RUBY_PLATFORM !~ /cygwin|mswin|mingw|bccwin|wince|emx/)
end

Vagrant.configure(2) do |config|

  # Ssh
  config.ssh.insert_key       = true
  config.ssh.username         = 'vagrant'
  config.ssh.forward_agent    = true

  #VM
  config.vm.box           = "debian/jessie64"
  config.vm.hostname      = app[:appname] + '.dev'
  config.vm.network       'private_network', type: 'dhcp'

  config.vm.synced_folder '.', '/vagrant',
    type: 'nfs',
    mount_options: ['nolock', 'actimeo=1', 'fsc']

  # Vm - Provider - Virtualbox
  config.vm.provider 'virtualbox' # Force provider
  config.vm.provider :virtualbox do |virtualbox|
    virtualbox.name   = app[:appname]
    virtualbox.memory = app[:box_memory]
    virtualbox.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    virtualbox.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
  end

  # Vm - Provision - Dotfiles
  for dotfile in ['.ssh/config', '.gitconfig', '.gitignore', '.composer/auth.json']
    if File.exists?(File.join(Dir.home, dotfile)) then
      config.vm.provision dotfile, type: 'file', run: 'always' do |file|
        file.source      = '~/' + dotfile
        file.destination = '/home/' + config.ssh.username + '/' + dotfile
      end
    end
  end

  verbosity_arg = if defined? ansible_verbosity then ansible_verbosity else '' end
  if host_box_is_unixy?
    config.vm.synced_folder "./", "/vagrant" + app[:appname],
      type: "nfs",
      mount_options: ['nolock', 'actimeo=1', 'fsc']
    config.vm.provision :ansible do |ansible|
      ansible.playbook = 'provisioning/site.yml'
      ansible.extra_vars = app
      ansible.verbose = verbosity_arg
    end
  else
    config.vm.synced_folder "./", "/vagrant" + app[:appname], type: "smb"
    extra_vars_arg = '{' + app.map{|k,v| '"' + k.to_s + '":"' + v.to_s + '"'}.join(',') + '}'

    config.vm.provision :shell, :inline => <<-END
set -e
if ! which ansible-playbook ; then
  echo "Windows host environment: the devbox will install Ansible and self-provision itself" >&2
  sudo apt-get update
  sudo apt-get -y install python-software-properties
  sudo add-apt-repository ppa:ansible/ansible
  sudo apt-get update
  sudo apt-get -y install ansible
fi
PYTHONUNBUFFERED=1 ansible-playbook --connection=local -i "[default] $(hostname)," \\
--extra-vars='#{ extra_vars_arg }' #{ verbosity_arg.empty? ? '' : '-'+ansible_verbosity } /vagrant/provisioning/site.yml
END
  end

  # Plugins - Landrush
  if Vagrant.has_plugin?('landrush')
    config.landrush.enabled            = true
    config.landrush.tld                = config.vm.hostname
    config.landrush.guest_redirect_dns = false
  end

end
