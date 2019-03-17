---
title: "Building my own 4G LTE Router"
excerpt: "In this project I'll describe how I built my own Router with 4G LTE Internet access, DHCP and DNS server with content filtering with a Raspberry Pi and a 4G Mobile Broadband Dongle"
permalink: /home-router-lte/
header:
  overlay_image: /assets/images/network_cables.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: You need admin rights to view this
categories:
  - Sysadmin
tags:
  - raspberrypi
  - router
  - lte
  - wifi
  - sysadmin
toc: true
---

# Project description

Recently I needed to find an alternative to my Broadband ADSL due to various connection issues and slow performance (lower than 4 Mbps). The only viable alternative is an LTE connection, so I choose to go with a 4G Mobile Broadband Dongle.

As I have my own private wired/wifi network, there is a need to have something more sophisticated than just plugging the dongle into a single PC! So my recipe is to set up my Raspberry Pi with the USB dongle as a Router.

## Network Scheme

![My Network Scheme]({{ "/assets/images/network.svg" }})

### What I have

* Raspberry Pi 3 Model B (Red)
* Tp-Link RE450 Access Point / Range Extender (Blue)
* Switch Gigabit Ethernet (Blue)

### What I need

* LTE Mobile Broadband Dongle (Violet)
* LTE External Antenna

The 4G LTE dongle I've used is manufactured by ZTE (MF823). The dongle is directly recognized as an Ethernet port `usb0`. A web interface is provided in order to get information about the connection, to enter the pin code, ...

## Address Space

Because I'm building a private network I'm using the following IPv4 address ranges (Class C network):

* `192.168.0.0/24` for `usb0` interface (the LTE Dongle one)
* `192.168.1.0/24` for `eth0` interface (the rest of the network)

# Configure everything

The following guide applies to the Raspbian operating system.

## Setup the dongle and access the LTE network

First check if the LTE dongle is recognized:

```bash
$ lsusb
... ZTE WCDMA Technologies MSM
```

The dongle should be recognized as a network card, providing an interface for transmitting Ethernet frames, and as a mass storage device (to access the Memory Card content).

```bash
$ ifconfig
usb0    Link encap:Ethernet HWaddr XX:XX:XX:XX:XX:XX
        inet addr:192.168.0.185 Bcast:192.168.0.255 Mask:255.255.255.0
        ...
```

The dongle will be listed as `usb0` and the interface should get an ip address in the `192.168.0.0/24` from the dongle DHCP.

To configure the dongle connection, connect it to a PC (I've used my Mac) with the SIM card installed, the dongle is provided with a led indicator which have three states:

* Red: the dongle is booting
* Blu: boot complete and ready to connect
* Green: connected to Broadband

Fire up a Web Browser and go to `http://192.168.0.1/` (the gateway address). You should be presented with a little configuration site, change or set the correct APN value depending on the carrier parameters.

Assuming the dongle is successfully connected to the internet (green light), test if the connection is working: just ping Google!

**Bonus fact**: the dongle is a Linux-like machine that run BusyBox, you can tweak the internal network parameters following [this guide](https://my-router.blogspot.com/2015/09/zte-mf823-4g-change-ip-of-modem-and-get.html).

## Configure the network

Assign a static IP address on the internal network with `sudo vi /etc/network/interfaces`

```text
# The loopback network interface
auto lo
iface lo inet loopback

# The internal network interface
auto eth0
iface eth0 inet static
address 192.168.1.1
netmask 255.255.255.0

# The dongle network interface
auto usb0
iface usb0 inet dhcp
```

### DHCP for internal network

See <a href='#dns-sinkhole'>DNS Sinkhole</a> on how to install and configure the DHCP server, I report here the values:

* Range of IP addresses: `192.168.1.50 192.168.1.150`
* Gateway address: `192.168.1.1`
* Broadcast address: `192.168.1.255`
* Subnet mask `255.255.255.0`
* DNS address: `192.168.1.1` (the sinkhole!)
* DHCP Lease Time: `24 hours`
* Static Leases: list devices which must have a static assigned address (possibly below `192.168.1.50`)

### Internet connection

To share the 4G LTE connection with the internal network devices, the traffic must be routed between `eth0` and `usb0`. The tool needed is the Linux firewall: iptables. The firewall forwards connections from the internal network to the 4G LTE network and vice-versa.

Install with:

```bash
sudo apt-get install iptables
```

First enable IPv4 forwarding: edit the file with `sudo vi /etc/sysctl.conf` and uncomment the line:

```text
net.ipv4.ip_forward=1
```

to enable IP forwarding immediately:

```text
sudo sysctl -w net.ipv4.ip_forward=1
```

To enable Network Address Translation, NAT and forwarding:

```bash
sudo iptables -t nat -A POSTROUTING -o usb0 -j MASQUERADE
sudo iptables -A FORWARD -i usb0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o usb0 -j ACCEPT
```

to save these firewall rules:

```bash
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

to reload these rules at the boot time, create a file `sudo vi /etc/network/if-up.d/iptables` with the following contents:

```sh
#!/bin/bash
/sbin/iptables-restore < /etc/iptables.ipv4.nat
exit 0
```

and make it executable `sudo chmod +x /etc/network/if-up.d/iptables`.

### DNS Sinkhole

A DNS sinkhole, is a DNS server that gives out false information, to prevent the use of a domain name.

The best implementation that works smoothly on the Raspberry Pi is **Pi-hole**.

To install it:

```bash
curl -sSL https://install.pi-hole.net | bash
```

Then use the Web Interface Dashboard to configure upstream DNS servers (like Cloudflare DNS `1.1.1.1 1.0.0.1`) and enable DHCP server.

## Wireless Access Point

The extender can work as an access point, transforming the wired network to a wireless one.

To select this operating mode is as simple as clicking _Mode_ in the top right corner of the page. Select _Access Point_ and click _Save_. The extender will reboot and switch to Access Point mode. Connect the extender to the switch via an Ethernet cable and configure it broadcasts the signal in the same network class.

# Run Kodi on the Rpi

The previous purpose of my Raspberry was to run [Kodi](https://kodi.tv/), it can be installed in many operating systems, including Raspbian but there are better choices that are more optimized while remaining Debian-based.

I chose to use [OSMC](https://osmc.tv/), which is based on Debian 9.5 Stretch and has all Debian packages (for the Raspberry architecture obviously).

## Changes for OSMC

OSMC is different from Debian in some respects such as networking and startup scripts. Accessing the command line is described [here](https://osmc.tv/wiki/general/accessing-the-command-line/).

### Network configuration

The file `/etc/network/interfaces` doesn’t exist, OSMC uses a _Connection Manager_ as to handle connections rather than the traditional networking service.

The [ConnMan documentation page](https://01.org/connman/documentation) states that ConnMan contains a very useful _command line tool_, `connmanctl`, which can be used to configure network interfaces.

Each network interface is presented as a _service_ identified with `<technology type>_<mac address>_<other info>`:

```bash
$ connmanctl services
*AR Wired   ethernet_abcdef012345_cable
*AR Wired   ethernet_abcdef678901_cable
```

With the `ifconfig` command identify each interface looking at the MAC address and configure them accordingly:

```bash
# Internal network eth0
$ connmanctl config ethernet_abcdef012345_cable ipv4 192.168.1.1 255.255.255.0

# Dongle network usb0
$ connmanctl config ethernet_abcdef678901_cable ipv4 192.168.0.100 255.255.255.0 192.168.0.1
# Or with DHCP
$ connmanctl config ethernet_abcdef678901_cable ipv4 dhcp
```

Optionally the [ConnMan service provisioning file](https://manpages.debian.org/jessie-backports/connman/connman-service.config.5.en.html) can be used.

### Startup scripts

To run a script or start a program when OSMC starts, the better approach is to use OSMC’s init system, called _systemd_.

First save the script file to reload iptables rules and make it executable:

```bash
$ cd /home/osmc
$ mkdir scripts
$ cd scripts
$ touch last.log
$ vi iptables-restore.sh
# write the script content
$ chmod +x iptables-restore.sh
```

The script content remains the same:

```sh
#!/bin/bash
echo $(date -u) > /home/osmc/scripts/last.log
/sbin/iptables-restore < /etc/iptables.ipv4.nat
exit 0
```

Create the _Unit File_ `sudo vi /lib/systemd/system/iptables-restore.service`:

```text
[Unit]
Description = Restore iptables rules
After = network.target network-online.target

[Service]
Type = simple
ExecStart = /home/osmc/scripts/iptables-restore.sh

[Install]
WantedBy = multi-user.target
```

Notify systemd that a new `iptables-restore.service` file exists by executing the following commands:

```bash
systemctl daemon-reload
systemctl enable iptables-restore.service
systemctl start iptables-restore.service
```

# Conclusions and future work

With this project I've had the chance to learn interesting things on Linux networking stack also I've enjoyed the pleasure of build and control my private network.

In the near future, I'd like to add a way to access my network from the outside (using a VPN service or similar) and fine tune the Raspberry firewall.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Building my own 4G LTE Router <a href="https://t.co/d0eIeHAba8">https://t.co/d0eIeHAba8</a> <a href="https://t.co/ifKkTM0VKn">pic.twitter.com/ifKkTM0VKn</a></p>&mdash; Filippo (@filippomito) <a href="https://twitter.com/filippomito/status/1030458050862297088?ref_src=twsrc%5Etfw">17 agosto 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>