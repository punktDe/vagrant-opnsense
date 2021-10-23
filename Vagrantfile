Vagrant.configure(2) do |config|

  # Which base box to use - this is FreeBSD 12.1 for now according to
  # https://github.com/opnsense/update
  $opnsense_box = 'punktde/freebsd-121-ufs'
  
  # Which OPNsense release to install
  $opnsense_release = '21.7'

  # IP address of the firewall in the host-only network
  $virtual_machine_ip = '192.168.1.1'

  # Configure folder sharing
  $vagrant_mount_path = '/var/vagrant'
  config.vm.synced_folder '.', '/vagrant', id: 'vagrant-root', disabled: true
  config.vm.synced_folder '.', "#{$vagrant_mount_path}", :nfs => true, :nfs_version => 3

  # Enable SSH keepalive to work around https://github.com/hashicorp/vagrant/issues/516
  config.ssh.keep_alive = true

  # Configure proper shell for FreeBSD
  config.ssh.shell = '/bin/sh'

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search
  config.vm.box = $opnsense_box

  # Create a private network, which allows host-only access to the machine
  # using a specific IP
  config.vm.network 'private_network', ip: $virtual_machine_ip, auto_config: false

  # Customize build VB settings
  config.vm.provider 'virtualbox' do |vb|
    vb.memory = 4096
    vb.cpus = 1
  end

  # Transfer config file snippets into VM
  config.vm.provision "file", source: "files", destination: "files"

  # Bootstrap OPNsense
  config.vm.provision 'shell', inline: <<-SHELL

    # Download the OPNsense bootstrap script
    fetch -o opnsense-bootstrap.sh https://raw.githubusercontent.com/opnsense/update/master/src/bootstrap/opnsense-bootstrap.sh.in

    # Remove reboot command from bootstrap script
    sed -i '' -e '/reboot$/d' opnsense-bootstrap.sh

    # Start bootstrap
    sh ./opnsense-bootstrap.sh -r #{$opnsense_release} -y

    # Set correct interface names so OPNsense's order matches Vagrant's
    sed -i '' -e 's/mismatch0/em1/' /usr/local/etc/config.xml
    sed -i '' -e 's/mismatch1/em0/' /usr/local/etc/config.xml

    # Remove IPv6 configuration from WAN
    sed -i '' -e '/<ipaddrv6>dhcp6<\\/ipaddrv6>/d' /usr/local/etc/config.xml

    # Remove IPv6 configuration from LAN
    sed -i '' -e '/<ipaddrv6>track6<\\/ipaddrv6>/d' /usr/local/etc/config.xml
    sed -i '' -e '/<subnetv6>64<\\/subnetv6>/d' /usr/local/etc/config.xml
    sed -i '' -e '/<track6-interface>wan<\\/track6-interface>/d' /usr/local/etc/config.xml
    sed -i '' -e '/<track6-prefix-id>0<\\/track6-prefix-id>/d' /usr/local/etc/config.xml

    # Enable SSH by default
    sed -i '' -e '/<group>admins<\\/group>/r files/ssh.xml' /usr/local/etc/config.xml

    # Allow SSH on all interfaces
    sed -i '' -e '/<filter>/r files/filter.xml' /usr/local/etc/config.xml

    # Do not block private networks on WAN
    sed -i '' -e '/<blockpriv>1<\\/blockpriv>/d' /usr/local/etc/config.xml

    # Create XML config for Vagrant user
    key=$(b64encode -r dummy <.ssh/authorized_keys | tr -d '\n')
    echo "      <authorizedkeys>${key}</authorizedkeys>" >files/vagrant2.xml
    cat files/vagrant[123].xml >files/vagrant.xml

    # Add Vagrant user - OPNsense style
    sed -i '' -e '/<\\/member>/r files/admins.xml' /usr/local/etc/config.xml
    sed -i '' -e '/<\\/user>/r files/vagrant.xml' /usr/local/etc/config.xml

    # Change home directory to group nobody
    chgrp -R nobody /usr/home/vagrant

    # Change sudoers file to reference user instead of group
    sed -i '' -e 's/^%//' /usr/local/etc/sudoers.d/vagrant

    # Reboot the system
    shutdown -r now

  SHELL
end
