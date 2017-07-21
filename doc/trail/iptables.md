# Setting up the iptables firewall on Linux

[iptables](https://en.wikipedia.org/wiki/Iptables) is a Linux based userspace program that allows configuration of the Linux kernel firewall.

To avoid lockout scenarios, the management of IPtables is disabled initially, however, if you followed the trails correctly then the required exceptions will have already been placed and you can safely activate the firewall by adding to hiera:

```
r_profile::linux::firewall::ensure: 'managed'   # activate the firewall
r_profile::puppet::master::open_firewall: true  # open the firewall on the master
```

Alternatively, you can deactivate the firewall by setting the key to 'disabled' - assuming you haven't just completely destroyed networking if the firewall was previously enabled...
