openwrt-fakewan
===============

Assign your WAN IP to a device on your LAN

# Dependencies

Developed against OpenWRT 14.07 "Barrier Breaker" with custom build options:

	CONFIG_KERNEL_NAMESPACES=y
	CONFIG_KERNEL_NET_NS=y

Depends on packages:

	ip-full
	kmod-veth
	snmp-utils (optional)

# Installation

**WARNING:** A mistake can lock you out of your router!  Be prepared in case
that happens: Backup your configs; have a firmware image and tftp client
ready just in case; temporarily enable luci/ssh access via the WAN port;
enable admin access via wireless, etc.

### Install the files

	/etc/init.d/fakewan
	/etc/config/fakewan
	/etc/udhcpc.user

### Configure /etc/config/fakewan

* Set slave_mac to your slave device's WAN interface MAC address.
* Change slave_ifname if your LAN interface is different.
* Change slave_vlan, veth0_ip, veth1_ip if necessary.

Test run before enabling:

	/etc/init.d/fakewan start

Confirm the interfaces were created:

	ifconfig fakewan0
	ip netns exec fakewan ifconfig fakewan1
	ip netns exec fakewan ifconfig eth0.101

If everything is OK so far, and you haven't locked yourself out:

	/etc/init.d/fakewan enable
	reboot

Verify again the interfaces were set up correctly after reboot.

### Configure the VLAN in the luci menu "Network -> Switch"

**WARNING:** Temporarily enable luci/ssh access via WAN and/or
wireless, in case you lock yourself out via the LAN switch!

Make sure "Enable VLAN functionality" is checked.

Example config (assumes the slave device is plugged into Port 3):

	VLAN ID    Port 0    Port 1    Port 2    Port 3    CPU
	---------  --------  --------  --------  --------  --------
	1          untagged  untagged  untagged  off       tagged
	101        off       off       off       untagged  tagged

Note: If VLANs were not enabled previously, you will need to update
your LAN bridge to add the VLAN interface (eth0.1) and remove the
physical interface (eth0)  Hopefully you paid attention the warnings
and enabled administration via WAN or wireless!

### In the luci menu "Network -> Interfaces", create a new interface:

	Name:
		(whatever)
	Protocol:
		Static address
	Cover the following interface:
		fakewan0
	IPv4 address:
		192.168.101.1 (must match the veth0_ip config option)
	IPv4 netmask:
		255.255.255.0 (must match the veth_netmask option)
	DHCP Server:
		disabled

### Configure the firewall zone as needed

Any ports you want forwarded to the slave device should forward to 192.168.101.2 (veth1_ip)

Reboot and re-check that everything got set up correctly.

### Verify the virtual network can route outside:

	ip netns exec fakewan ping -c 4 8.8.8.8

### Connect the slave device WAN port to the designated fakewan port

* Did it get assigned the correct IP address by DHCP?
* Do DNS lookups work?
* Does it have Internet access?
* Do your port forwards work?
