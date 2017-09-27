==================================
Hackeriet's network infrastructure
==================================

This repository contains treasures.

Overview
=========

map_haus_nett - a map of the landscape

Gateway
=======

DEPRECATION NOTE: look in the "moon" directory.
There you will find much more extensive documentation
about the new setup, in effect since 2017-09-27.

The gateway is configured in the following files:

Firewall
--------

``/etc/rc.firewall``

Traffic shaping
---------------

``/usr/local/sbin/wshaper.htb``

called from the firewall

DHCP and DNS
------------

``/etc/dnsmasq.conf``


Check your work
---------------
iftop -i eth1 -m 20M -P


switch_map_haus
===============

what connects where on what port and what vlan


eXtreme
=======

extreme-config-template.txt - how to configure eXtreme switches

Quick tutorial::
 - show which VLAN a port has: ``show ports 9 vlan info``
 - list all vlans on the switch: ``show vlan``
 - set VLAN on a port: ``configure vlan VLANNAME add ports 9 untagged`` (might need to ``configure vlan OLDVLAN delete ports 9`` first.)
 - save config to nvram: ``save``

HP Procurve
===========

We have a few managed HP Procurve switches. They usually have "manager" as the SSH username.

* https://h10145.www1.hp.com/downloads/DownloadSoftware.aspx?SoftwareReleaseUId=16635&ProductNumber=J4900C
* http://services.geant.net/cbp/Knowledge_Base/Campus_Networking/Documents/gn3-na3-t4-cbpd111.pdf 
* http://evilrouters.net/2009/02/02/upgrading-procurve-firmware-via-tftp/

Splash
======

Page delivered to those we provide internet to.


