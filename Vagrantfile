Vagrant.configure(2) do |config|

  # Which base box to use
  $opnsense_box = 'punktde/freebsd-121-ufs'

  # User settable  box parameters here
  $vagrant_mount_path = '/var/vagrant'
  $virtual_machine_ip = '192.168.1.1'

  # Disable folder sharing
  config.vm.synced_folder '.', '/vagrant', id: 'vagrant-root', disabled: true

  # Enable SSH keepalive to work around https://github.com/hashicorp/vagrant/issues/516
  config.ssh.keep_alive = true

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = $opnsense_box

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network 'private_network', ip: $virtual_machine_ip, auto_config: false

  # Customize build VB settings
  config.vm.provider 'virtualbox' do |vb|
    vb.memory = 4096
    vb.cpus = 1
  end

  # Bootstrap OPNsense on stock FreeBSD box
  config.vm.provision 'shell', inline: <<-SHELL

    # do OPNsense installation
    fetch https://raw.githubusercontent.com/opnsense/update/master/bootstrap/opnsense-bootstrap.sh
    sh ./opnsense-bootstrap.sh -y

    # reboot system
    shutdown -p now

  SHELL
end
