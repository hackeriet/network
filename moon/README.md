## Moon, a router gateway for Hausmanns

#### PREFACE
Some facts:

* 2017-09-26: on this date we had to install a new router / gw for Hackeriet and Hausmania, as the previous one died.
* It's a Debian stretch setup, using two network interfaces
* We tried to keep the setup as simple as possible, we had enough of complicated and cutting edge setups that people don't understand and
(obviously) don't dare even try to manage. KISS: "Keep It Simple Stupid". It's a blessing, a love meesage to you, my friend.

## What do i need to get a machine on her legs

One can do a lot of nifty work on a router / gateway, but what is the minimum i need to get one to function?
Here is the stricly necessary stuff to make it work. Ensure these configurations are in place in the corresponding configuration files.

### /etc/network/interfaces

Packages to install: vlan

We have two interfaces, one is the uplink (eno1) and the other one (eno2) is connected to a switch trunking the necessary virtual lans.
In order to trunk vlans to a switch, the "vlan" package has to be installed, and then virtual lans can be added in order to create additional
subnet segments. This is useful in order to isolate different environments, for example the net where a lot of unexperienced and vicious freaks
cause havok per default in their life, from the one in which experienced vicious freaks try to avoid havok in their life. Or something like that.

The uplink (eno1) is assigned a public ip, and has a static ip configured in order to come cleanly up after a reboot. I won't use more
details here, just look at the interfaces file

The downlink (eno2), will work as a gateway for all the subnets. I provide the two snippets that are necessary for the hackeriet subnet inline here.
The first snippet is the physical eno2 interface, nothing interesting here, just a starting point to add vlans. Still necessary.

```
auto eno2
iface eno2 inet static
address 10.0.1.2
netmask 255.255.255.0
```

The second snippet adds a .3 to the interface eno2, because this is a virtual one, referring to the physical one.

```
auto eno2.3
iface eno2.3 inet static
address 10.10.3.1
netmask 255.255.255.0
up vconfig set_flag eno2.3 1 1
```

Observe as well:

* The address reflects the geometry of the net, from the netmask we know is a /24,
* the address itself is a .1 because this interface will be the gateway for that subnet,
* in the last line we add a vconfig command in order to trunk the traffic for this subnet
through the spanning tree protocol. To explain it in an oversimplified way, we tell the switch
downstream that we are sending traffic for a certain subnet over the same cable we use for
other subnets, so traffic for the different subnets over the same cable is marked with the
relative subnet or, to say it properly, is "tagged".

#### Really? I am lost! Tell me more.

Developers! We have a switch downstream from the second interface, and we send traffic tagged with the different vlans.
On the switch (that has to have the proper spanning tree protocol functionality) that receives the cable from the second interface of
our gateway on a port in which we tag all the vlans. That means, in our setup we need to have:

* An stp enabled switch
* The corresponding vlans we defined in the interfaces file configured on the switch
* The correspoding vlans tagged on the switch port connected to the gateway
* The corresponding vlans untagged on the ports we use to connect the physical client hosts to

..so we can distribute DHCP to the different ports according to the vlan they are assigned to.

We could of course have used a machine with more ethernet ports, configured them as the first snippet,
and just added some stupid switches downstream. Good switches do so much more though, like providing
protection from different evil things your users will try doing at some point, or traffic shaping, and of course the setup
using vlans can always provide all the vlans selectively on the ports of all the switches connected to your tree.
Just by using a combination of "untagged" and "tagged" traffic.

As a rule of thumb:

* tagged traffic is for switches with stp enabled
* untagged traffic is for client hosts

Now do your homework.

###  /etc/dnsmasq.conf

Packages to install: dnsmasq

Once the subnets are in place, we need to have a DHCP server answering there.
Dnsmasq does that, and some other things, like adding some internal dns records and so on.
We don't use most part of the features, what we like about it is that is very simple to setup.
Check that dnsmasq is installed. Once that is done, you can use the dnsmasq.conf provided.

Note that the necessary lines for answering to DHCP on the matching vlans we added in /etc/network/interfaces
look like this:

```
dhcp-range=hackeriet,10.10.3.50,10.10.3.250,255.255.255.0,6h
```

Where you have an ip range you use as a pool for DHCP, a netmask, and a TTL.
Every vlan should have one of these.
You don't need much more actually, but those one are necessary.
Ensure that the dnsmasq comes up at reboot, systemctl enable dnsmasq should do the trick.

###  /etc/sysctl.conf
Subnets are in place, we have a DHCP, but we also need to forward traffic on the way out.

* These are a must have:

```
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

Other lines from the default can be added, check the conifguration file, we don't need to discuss more of them here.
Activate the edited sysctl.conf running "sysctl -p", these are now persistent, and will come up at reboot.

### Iptables rules

Packages to install: iptables-persistent

Last but not least, we need to reflect the previous step also in the firewall rules.
We actually need only one, important line, and that looks like this:

```
iptables --table nat --append POSTROUTING -o eno1 -j MASQUERADE
```

We are fed up of having custom scripts laying around, so we do simply two things:

* run the previous iptables command
* run:

```
iptables-save > /etc/iptables/rules.v4
```

..so that the rules are persistent, and come up again at reboot.
Once you came all the way here, you have a functioning router gateway. Congrats. You have a clients.

## More stuff (but not strictly necessary)

Ensure that the following is in place:

* ntp is installed.
* /etc/localtime points to Oslo.
* password authentication is turned off and your public ssh-key is where it is supposed to be.
* fail2ban is installed and guards ssh from brute force attacks (because of course, you will turn password authentication back on at some point, DEV).
* unattended-upgrades is installed ( if you have read all this document, i can't imagine seeing you keeping your machine security updated ).

### More iptables rules?

In the old days we had a more basic infrastructure, and we had much less bandwidth than we do today. We also had worse switches, and in some extent,
worse people behind their client machines. The use of cable internet has dropped in the house, most people use some form of poor wireless access point.
Access points per default have their own NAT and some additional client isolation, and glossy crap of course...
Today some of the protection we implemented in the firewall rules is covered by the switches we use.
However, for the record, in the old days we used an rc.firewall script that per default dropped all traffic, while allowing only the necessary traffic.
We are now going to do the opposite, because we don't want to deviate from an installation that is minimal and easy to mantain.
We will see if this is a good idea or not.

