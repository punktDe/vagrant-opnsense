Bootstrap an OPNsense plugin development environment in Vagrant
===============================================================

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

Provision the VM
----------------
````
git clone git@github.com:punktDe/vagrant-opnsense.git
cd vagrant-opnsense
vagrant up
````
This will automatically
1. download a plain FreeBSD 12.1 Vagrant box provided by [punkt.de infrastructure](https://infrastructure.punkt.de/).
2. boot the VM.
3. convert the VM into an OPNsense installation with the [bootstrap](https://github.com/opnsense/update/) method.
4. reboot the resulting VM.

Should you need to repeat this step from the start you can always
```
vagrant destroy
vagrant up
```

Connect via your browser
------------------------
![Browser](img/browser.png)

Congratulations! You have a working OPNsense installation in Vagrant/Virtualbox.
Now navigate through the initial setup wizard.

Next steps
----------
You should install the `os-virtualbox` plugin so you can cleanly shutdown and startup
the system.

Then probably register an SSH public key for the root user, enable root login via SSH and
change the OPNsense menu to a shell for root.

Enjoy!
