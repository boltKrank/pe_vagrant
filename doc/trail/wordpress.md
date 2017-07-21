# Setting up A LAMP server with WordPress

[WordPress](https://wordpress.com/) is a [PHP](https://www.php.net/) based [CMS](https://en.wikipedia.org/wiki/Content_management_system) that works with a variety of databases and web servers.  For this trail, we've picked [MySQL](http://www.mysql.com/) database and [Apache HTTPd](https://httpd.apache.org/) web server for simplicity.

Add the following to hiera:

```yaml
r_profile::web_service::apache::lb: "192.168.13.110" # replace with IP address of load balancer
r_profile::web_service::apache::open_firewall: true

r_profile::lb::haproxy::listeners:
  apache:
    collect_exported: true
    ipaddress: '0.0.0.0'
    ports: '80'

```

## Load balancer

/site/profile/lib/facter/source_ipaddress_targets.txt

```
192.168.13.110
```

* In order to resolve outgoing address to use to reach the load balancer, the load balancer IP Address needs to be present in `source_ipaddress_targets.txt`.  This file will be distributed by [pluginsync](https://docs.puppet.com/puppet/4.8/plugins_in_modules.html#auto-download-of-agent-side-plugins-pluginsync).  The easiest place for this file to live at is `./site/profile/lib/facter/source_ipaddress_targets.txt` in your control repo.  This file will be synced before each facter run, so it is not necessary to run puppet multiple times to setup the facts.
* Add the `apache` stanza to `r_profile::lb::haproxy::listeners`.  Note that you must _merge_ this into any existing definition, otherwise you will _replace_ it.
