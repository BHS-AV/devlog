---
layout: post
title: Network Configuration
subtitle: Connecting our car
gh-repo: bhs-av/software
tags: [networking]
comments: false
---

For our car to function properly it is important that we develop connections to the car as well as allow the car to connect to the internet. The car needs to connect to the internet in order to get updates and software while also connecting to a local computer so that it can be controlled.

# Old Configuration
Before our current implementation we had a monitor that we would attach the Jetson to. This process was incredibly slow because the monitor would take a while to detect the Jetson. Furthermore there were some display issues with the Jetson where it would turn off without an video output. In addition to these, we had to plug in and unplug a keyboard and mouse each time we wanted to change even a single line of code. Over the last year, we lost a lot of time changing back and forth, so our new implementation helps streamline the development process.

# New Configuration
Our new configuration makes use of two separate networks on the RACECAR. First the wireless network on the Jetson connects to a normal Wi-Fi network, while another Local Area Network (LAN) is created using a router built into the RACECAR.

## Wi-Fi Network
The first and most important connection we have to make is through a Wi-Fi network. This will allow the Jetson to connect to the internet to get updates and install new software. Without connection to the internet, it would be impossible to make new updates or link with our GitHub page in order to clone new code onto the Jetson.

### Setting Up Wi-Fi
Wi-Fi is set up normally using the Network Manager built into Ubuntu.
- Open the **Network Manager** and click Add.
- Add a Wi-Fi network and select the network that has access to the internet. By default, the Jetson will use this network to access the Internet.
- Check that the Jetson has access to the Internet.

## Local Area Network (LAN)
The LAN will allow any laptop with the requisite software to connect to the Jetson and control it. By default, the Jetson does not support having two networks at once so we have to configure it to use the LAN for data transfer and the Wi-Fi network to access the Internet.

### Setting up the LAN
LAN is set up in a similar way to the Wi-Fi network.
- Setup the router by assigning it an SSID and a password.
- Connect the Jetson to the router using an ethernet cable.
> The Jetson will not connect to the LAN without an ethernet cable to the router.

- Open the **Network Manager**.
- Select **Edit Connections**.
- Select the network connection labeled **Wired Connection** that corresponds to the ethernet port that is connected to the router and choose **Edit**.
- Select the **IPv4** settings tab and change the method **Automatic (DHCP)** to **Disabled**. This will prevent the router from reassigning the Jetson's IP as well as create a persistent proxy to the LAN.
- Save the connection and reopen **Network Manager**.
- Add a new connection, and change the mode to **Ethernet**
- Assign the connection a name. This connection will act as a switch between the router and the Wi-Fi network, allowing the Jetson to access both at the same time.
- In **IPv4** settings, change the method to **Manual**.
- Select **Add** Addresses and set the address to any value.
> The router will assign the Jetson an IP with the form `10.x.y.z`, `172.x.y.z`, or `198.168.y.z`. When adding an address use one with the same form as assigned by your router (can be found using `ifconfig`).
- Set the **Netmask** to `255.255.255.0` and leave **Gateway** blank.
- Fill the DNS server field with `10.0.0.10`.
- Save the network configuration.

# Conclusion
After completing both sections above, the Jetson will now be able to access both the internet and the LAN through the router. This separation allows us to control the settings of the LAN and not deal with the firewalls implemented by our school network.

In addition to providing us more control over the Jetson, this setup allows us to use a SSH setup to remotely access the Jetson. In the next post we will discuss our SSH implementation and how we use X11 forwarding to make it easy to access the Jetson from any laptop.
