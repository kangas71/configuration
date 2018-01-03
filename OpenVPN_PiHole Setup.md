# Pi-Hole DNS AD Blocking via OpenVPN.

This is a quick and dirty guide on setting up ad blocking via openVPN and Pi-Hole. Once you set this up, you'll enjoy a secure VPN connection with ad blocking to your home network while connected to open WiFi networks.

### Step 1.
Install openVPN on an Ubuntu Server or any of the supported servers listed on openVPN's's website. Digital Ocean has a fantastic how-to guide, so I'm not going to re-write all this. Just follow their guide ...

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04

Note: if your running more than one VPN connection on your home LAN you will want to verify the IP subnets. If both VPN's are set to run on 10.8.0.x subnets, which is the default for openVPN, you'll want to update one of them to run within a different IP subnet. Such as 10.9.0.x for example. You can adjust this when your in the `/etc/openvpn/server.conf` (*Note: change 'server.conf' to whatever you named your openVPN server when following Digital Ocean's guide*) file. Look for the "Configure server mode and supply a VPN subnet" section and adjust the following from the default "server 10.8.0.0 255.255.255.0" to ...

```
server 10.9.0.0 255.255.255.0
```

### Step 2.
Install pi-hole on an Ubuntu Server or any of the supported servers listed on pi-hole's website. You can install it on the same server as openVPN or a second server. I have installed it on a second server that is a virtual machine on my hypervisor. Again, the how-to guides on installing pi-hole is good as well. Here is the guide ...

https://github.com/pi-hole/pi-hole

*Note: In pi-hole settings you can add entries for Upstream DNS Servers. I like to use Quad9's Internet Security & Privacy DNS service.*

https://www.quad9.net/#/#setup-quad9

![alt text](screenshots/pi-hole_DNS_v1.1.png "pi-hole_DNS View")

### Step 3.
Allow internal static routing from your openVPN server into the resources on your LAN. By default this is not enabled in the openVPN config files. To do this edit your configuration file in vi or nano `/etc/openvpn/server.conf`. Look for the "Push routes to the client to allow it to reach other private subnets behind the server" section and enable your push route to your LAN subnet IP address ...

```
push "route 192.168.0.0 255.255.255.0"
```

### Step 4.
Setup a static route in your router. This will tell the router where to route your 10.8.0.x (VPN) traffic. Every brand of router is different on how to set this up. Here is an example of what it should look like ...

| Network/Host IP |    Netmask    |   Gateway   | Interface |
|:---------------:|:-------------:|:-----------:|:---------:|
|    10.9.0.0     | 255.255.255.0 | 192.168.0.3 |    LAN    |

Network/Host IP is the IP address of your new VPN subnet and the Gateway (*update this IP address to what your static IP address is on your openVPN server*) is the IP address for your openVPN server. This tells your router to route all 10.9.0.x traffic via the VPN gateway at 192.168.0.3.

### Step 5.
Edit your configuration file on your openVPN server to point to your pi-hole DNS IP address. To do this just vi or nano your `/etc/openvpn/server.conf` file and look for the "Certain Windows-specific network settings can be pushed to clients, such as DNS or WINS server addresses." section and adjust the follow line of code ...

```
push "dhcp-option DNS 192.168.0.2"
```

Now test everything and you should be good to go!
