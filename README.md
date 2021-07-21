Bootstrap an OPNsense development environment in Vagrant
========================================================

Requirements
------------

* A system capable of running VirtualBox
* [Vagrant](https://www.vagrantup.com)
* [VirtualBox](https://www.virtualbox.org)

Preparation
-----------

Create a host-only network in VirtualBox with an IP address in `192.168.1.0/24`
but **not** `192.168.1.1`. This is the address your OPNsense will use for the LAN
interface by default. Make sure DHCP is disabled on that interface.
![Host Network Manager](img/vboxnet-settings.png)

Selecting the OPNsense version
------------------------------

You can set the variable `$opnsense_release` to either `21.1` or `21.7` in [Vagrantfile](Vagrantfile)
to select the matching major release version. This requires destruction and re-provisioning of the box
if you want to change it after initial provisioning.

Provision the VM
----------------

```sh
git clone git@github.com:punktDe/vagrant-opnsense.git
cd vagrant-opnsense
vi Vagrantfile # adjust OPNsense version if desired
vagrant up
```

This will automatically

1. download a plain FreeBSD 12.1 Vagrant box provided by [punkt.de infrastructure](https://infrastructure.punkt.de/).
2. boot the VM.
3. convert the VM into an OPNsense installation with the [bootstrap](https://github.com/opnsense/update/) method.
4. adjust the configuration for this development environment - SSH will be enabled and permitted on all interfaces!
5. reboot the resulting VM.

Should you need to repeat this step from the start you can always

```sh
vagrant destroy
vagrant up
```

Connect via your browser
------------------------

![Browser](img/browser.png)

Use the default user and password of `root/opnsense`.

Congratulations! You have a working OPNsense installation in Vagrant/Virtualbox.
Now navigate through the initial setup wizard or skip it as instructed in the UI.

Connect via SSH
---------------

Use `vagrant ssh` to login. `sudo` will work without password.

Additional steps
----------------

* You should install the `os-virtualbox` plugin so you can cleanly shutdown and startup the system.
* Also disable the DHCP server on LAN.

Changing the LAN IP address
---------------------------

If you want to change the LAN network after initial deployment, e.g. because you use
`192.168.1.0/24` already, use these steps:

1. Change the IP address in the UI, save and apply. Use anything **but** the lowest address (.1)
   Keep a `/24` netmask. You will lose connectivity, of course.
2. Use `vagrant halt` to shutdown the VM. Vagrant connects via WAN, so this still works.
3. Edit `Vagrantfile` and change `$virtual_machine_ip` to your new value.
4. Start the VM with `vagrant up`. Vagrant will automatically create a matching host-only network
   and use the lowest address (.1) for your development system.
5. Use the new address to connect via browser once the VM is up and running.

---
Enjoy!
