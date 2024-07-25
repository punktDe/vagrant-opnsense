Vagrant.configure(2) do |config|

  #
  # General settings
  #

  $opnsense_box = 'punktde/freebsd-141-ufs' # Which base box to use
  $opnsense_release = '24.7'                # Which OPNsense release to install
  $virtual_machine_ip = '192.168.56.56'     # IP address of the firewall in the host-only network
  $vagrant_mount_path = '/var/vagrant'      # Shared path for development environment

  #
  # Box configuration
  #

  config.vm.synced_folder '.', '/vagrant', id: 'vagrant-root', disabled: true
  config.vm.synced_folder '.', "#{$vagrant_mount_path}", :nfs => true, :nfs_version => 3

  config.ssh.shell = '/bin/sh'
  config.ssh.keep_alive = true

  config.vm.box = $opnsense_box

  config.vm.network 'private_network', ip: $virtual_machine_ip, auto_config: true

  config.vm.provider 'virtualbox' do |vb|
    vb.memory = 4096
    vb.cpus = 1
  end

  #
  # Bootstrap OPNsense
  #

  config.vm.provision "file", source: "files", destination: "files"

  config.vm.provision 'shell', inline: <<-SHELL

    # Download the OPNsense bootstrap script
    fetch -o opnsense-bootstrap.sh https://raw.githubusercontent.com/opnsense/update/master/src/bootstrap/opnsense-bootstrap.sh.in

    # Remove reboot command from bootstrap script
    sed -i '' -e '/reboot$/d' opnsense-bootstrap.sh

    # Remove pkg unlock command from bootstrap script, which causes an error due to an upstream bug.
    # https://github.com/freebsd/pkg/issues/2278
    # This won't hurt even with the bug eventually fixed, because we don't have any locked packages.
    sed -i '' -e '/pkg unlock/d' opnsense-bootstrap.sh

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

    # Change OPNsense LAN IP addresses to VirtualBox compatible one
    sed -i '' -e "s/192\.168\.1\.1</#{$virtual_machine_ip}</" /usr/local/etc/config.xml

    # Change DHCP range to match LAN IP address
    lan_net=$(echo "#{$virtual_machine_ip}" | sed 's/\.[0-9]*$//')
    sed -i '' -e "s/192\.168\.1\./${lan_net}./" /usr/local/etc/config.xml

    # Enable SSH by default
    sed -i '' -e '/<group>admins<\\/group>/r files/ssh.xml' /usr/local/etc/config.xml

    # Allow SSH on all interfaces
    sed -i '' -e '/<filter>/r files/filter.xml' /usr/local/etc/config.xml

    # Do not block private networks on WAN
    sed -i '' -e '/<blockpriv>1<\\/blockpriv>/d' /usr/local/etc/config.xml

    # Reset shell of Vagrant user
    /usr/sbin/pw usermod vagrant -s /bin/sh

    # Create XML config for Vagrant user
    key=$(b64encode -r dummy <.ssh/authorized_keys | tr -d '\n')
    echo "      <authorizedkeys>${key}</authorizedkeys>" >files/vagrant2.xml
    cat files/vagrant[123].xml >files/vagrant.xml

    # Add Vagrant user - OPNsense style
    sed -i '' -e '/<\\/member>/r files/admins.xml' /usr/local/etc/config.xml
    sed -i '' -e '/<\\/user>/r files/vagrant.xml' /usr/local/etc/config.xml

    # Change home directory to group nobody
    chgrp -R nobody /home/vagrant

    # Change sudoers file to reference user instead of group
    sed -i '' -e 's/^%//' /usr/local/etc/sudoers.d/vagrant

    # Display helpful message for the user
    echo '#####################################################'
    echo '#                                                   #'
    echo '#  OPNsense provisioning finished - shutting down.  #'
    echo '#  Use `vagrant up` to start your OPNsense.         #'
    echo '#                                                   #'
    echo '#####################################################'

    # Shutdown the system
    shutdown -p now

  SHELL
end
