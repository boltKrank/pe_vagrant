# Setting up a monitor server

## General
Add the following to hiera:

```yaml
r_profile::monitor::nagios_monitored::nagios_server: "192.168.13.120" # replace with IP of monitor server
```

Add the nagios server IP address to `source_ipaddress_targets.txt`:

/site/profile/lib/facter/source_ipaddress_targets.txt
```
192.168.13.120
```

* This is need to allow us to work out what interface to ping-test hosts with for hosts on multi-honed networks
* In order to resolve outgoing address to use to reach the monitor server, the load balancer IP Address needs to be present in `source_ipaddress_targets.txt`.  This file will be distributed by [pluginsync](https://docs.puppet.com/puppet/4.8/plugins_in_modules.html#auto-download-of-agent-side-plugins-pluginsync).  The easiest place for this file to live at is `./site/profile/lib/facter/source_ipaddress_targets.txt` in your control repo.  This file will be synced before each facter run, so it is not necessary to run puppet multiple times to setup the facts.

## Firewall
```yaml
r_profile::web_service::apache::open_firewall: true
```

* You must open the firewall to be able to view the nagios webapp.

## Login to the nagios console
* Username `nagiosadmin`
* Password `changeme`

If using the Vagrant lab, you can use this link to login to the [Nagios Console](http://nagiosadmin:changeme@monitor.demo.internal/nagios/)

## Settings
```yaml
# change the nagiosadmin password
r_profile::monitor::nagios_server::password: 't0ps3cret123'
```

## Enable monitoring of the mysql service
By default the MySQL TCP service binds only to local host, so will show up as failed in nagios.  If you would like to monitor it properly, you could try opening up the port, like this:

```yaml
r_profile::database::mysql_server::override_options:
  mysqld:
    bind-address: '0.0.0.0'
```

This *would* have been a great way to demonstrate puppet's ability to change systems if it were not for https://tickets.puppetlabs.com/browse/FM-5705 -- so you will have to manually restart the service after changing the above settings and running puppet:
```shell
systemctl restart mariadb
```

A better workaround would be to use something like [nrpe](https://exchange.nagios.org/directory/Addons/Monitoring-Agents/NRPE--2D-Nagios-Remote-Plugin-Executor/details) to get into the machine and check status


## Things to do
* Look on the `Hosts` screen - each managed host shows up here
* Look on the `Services` screen - each managed service shows up here
* There's not really much to do on the Nagios trail, its more of a side-track to show that its easy to setup monitoring with puppet
* Adding monitoring to a customer's service might be a fun thing to do (or show the code they need to add)
* There is currently an error in the ping tests for `monitor.demo.internal` which appear related to using ping6 for the test against a link-local host.  Fixing this is left an an exercise for the user
