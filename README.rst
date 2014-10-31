==================================
Hackeriet's network infrastructure
==================================

This repository contains treasures.

Overview
=========

map_haus_nett - a map of the landscape

Gateway
=======

The gateway is configured in the following files:

Firewall
--------

`/etc/rc.firewall`

Traffic shaping
---------------

`/usr/local/sbin/wshaper.htb`

called from the firewall

DHCP and DNS
------------

`/etc/dnsmasq.conf`


Check your work
---------------
iftop -i eth1 -m 20M -P


switch_map_haus
===============

what connects where on what port and what vlan


eXtreme
=======

extreme-config-template.txt - how to configure eXtreme switches

Splash
======

Page delivered to those we provide internet to.


