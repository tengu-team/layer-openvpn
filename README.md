# Overview

This charm provides [OpenVPN Community VPN](http://openvpn.net/index.php/open-source).

This Charm installs and configures the VPN service and creates client certificates. What can you do with this Charm?

 1. Give remote users secure access to an internal network and let them use internal DNS servers.
 2. Secure remote users' communications by tunneling all their traffic through a secure connection.

# Administration

Deploy the application and you're ready to go!

```bash
juju deploy openvpn
```

Please note that this charm must be deployed on physical or virtual machines. This Charm does not work in LXC/LXD containers. Also note that changing the key settings will cause existing client configs to fail.

**Metrics**

This Charm exposes the number of connected clients using a juju metric.

```bash
$ juju metrics --all
UNIT        TIMESTAMP                METRIC     VALUE
openvpn/0   2016-11-27T15:05:25Z     users      1
```

You can find more detailed status information on the unit itself.

```bash
# On the openvpn unit
sudo cat /var/log/openvpn/openvpn-server1-status.log
```

## Configuration

- **push-dns** [`True`]: Set to False if clients shouldn't use the server's DNS settings.
- **push-default-gateway** [`True`]: Set to False if you want to use the VPN only for connections to servers in the private subnet. By default, ALL traffic will go over the VPN. Note that NetworkManager uses the VPN as default gateway regardless of server config. Use `openvpn` from the commandline to enable this behavior.
- **port** and **protocol**  [`443:tcp`]: `443:tcp` and `8080:tcp` have the least chance of being blocked by firewalls. `1194:udp` is the fastest.
- **key-*** : Information for key certificate. You don't actually need to change this.



# How do I get the OpenVPN client config file?

An OpenVPN client needs a config file to connect to the OpenVPN server. This Charm generates these config files for each client and puts them in `/home/ubuntu/<client-name>.ovpn`. You can download these config files using the Juju CLI. See the `clients` config option for more info.

```bash
juju scp openvpn/0:~/<client-name>.ovpn .
```

<a name="connect-howto"></a>
# How do I connect to the VPN?

The client config file works with any OpenVPN-compatible client on any OS. Use the instructions linked below or refer to the generic OpenVPN instructions for your OS.

- [Connect an Ubuntu Desktop.](#ubuntu-desktop-client)
- [Connect an Ubuntu Server.](#ubuntu-server-client)
- [Connect a MacOSX Desktop.](https://openvpn.net/index.php/access-server/docs/admin-guides-sp-859543150/howto-connect-client-configuration/183-how-to-connect-to-access-server-from-a-mac.html)


<a name="ubuntu-desktop-client"></a>
## Connect an Ubuntu Desktop

**Install OpenVPN client**

Install the OpenVPN network-manager integration. This will add the "VPN connections" menu in the network applet.

```bash
sudo apt install network-manager-openvpn-gnome
```

**Add VPN using config file**

1. Click the Network applet.
2. Choose `VPN connections > Configure VPN` as shown in the picture below.

<img src="https://raw.githubusercontent.com/tengu-team/layer-openvpn/master/files/documentation/networkmanager-applet.png" width="400">


3. Click *"Add"*.

<img src="https://raw.githubusercontent.com/tengu-team/layer-openvpn/master/files/documentation/add-vpn.png" width="400">


4. Scroll all the way down and click *"import a saved VPN configuration"*.

<img src="https://raw.githubusercontent.com/tengu-team/layer-openvpn/master/files/documentation/import-vpn-config.png" width="400">


5. Select the `.ovpn` config file, add the VPN, and connect using the network applet.

6. *[Optional] Regardless of server configuration, NetworkManager uses the VPN as default gateway, effectively sending ALL traffic over the VPN. If you set `push-default-gateway` to False and want NetworkManager to respect that setting, you need extra configuration on the client. Edit the VPN connection > IPv4 Settings > Routes...'.*

<img src="https://raw.githubusercontent.com/tengu-team/layer-openvpn/master/files/documentation/no-default-gateway-2.jpg" width="400">


7. *[Optional] Then mark "Use this connection only for resources on its network."*

<img src="https://raw.githubusercontent.com/tengu-team/layer-openvpn/master/files/documentation/no-default-gateway-3.jpg" width="400">

<a name="ubuntu-server-client"></a>
## Connect an Ubuntu Server

Use the following instructions to connect an Ubuntu server to the VPN.

```bash
sudo apt install openvpn
sudo openvpn --config <client-name>.ovpn
# Use the following command if you want to use the DNS settings that the OpenVPN server pushes
sudo openvpn --config <client-name>.ovpn --script-security 2 --up /etc/openvpn/update-resolv-conf --down /etc/openvpn/update-resolv-conf
```

# Known limitations

 - NetworkManager uses the VPN as default gateway regardless of server config. Follow steps 6. and 7. to disable this.
 - For cases where the VPN is not be the default gateway, and DNS settings are enabled, it is important to keep in mind that the clients will have two options for DNS nameservers: a public one (from the clients network) and a private one (from the network behind the VPN). The `openvpn` cli client will strictly use the private nameserver. Network Manager is a little bit smarter. Network Manager will send the DNS query to the public nameserver unless the url address is part of the search domain of the private network. This means that if the search domain on the private network is `example.com`, queries for `intranet.example.com` will be send to the private DNS server and queries for `www.google.com` will be send to the public DNS server. More information: https://bugs.launchpad.net/ubuntu/+source/openvpn/+bug/1211110/comments/50
 - If you use the VPN on Google Compute Engine with `push-default-gateway=False`, then traffic to GCE VM's will not go over the VPN by default. This is because each GCE VM is in a `255.255.255.255` network, so it has no idea which networks it has acces to, i.e. which networks it should push to VPN clients. You will need to manually add a route in the VPN clients if you want this to happen.

# Contact Information

## Bugs

Report bugs in the [layer-openvpn Github repo](https://github.com/tengu-team/layer-openvpn/issues).

## Authors

This software was created in the [IBCN research group](https://www.ibcn.intec.ugent.be/) of [Ghent University](https://www.ugent.be/en) in Belgium. This software is used in [Tengu](https://tengu.io), a project that aims to make experimenting with data frameworks and tools as easy as possible.

 - Merlijn Sebrechts <merlijn.sebrechts@gmail.com>
 - Images come from [TorGuard OpenVPN guide](https://torguard.net/knowledgebase.php?action=displayarticle&id=53) and AskUbuntu.
