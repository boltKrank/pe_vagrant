# Setting up a load balanced GeoServer

GeoServer is a Java-based [GIS](https://en.wikipedia.org/wiki/Geographic_information_system) that runs inside a Java Servlet container (we use [tomcat](http://tomcat.apache.org/)):

Add the following to hiera:

```yaml
r_profile::web_service::tomcat::lb: "192.168.13.110" # replace with IP address of load balancer
r_profile::web_service::tomcat::open_firewall: true


r_profile::lb::haproxy::listeners:
  tomcat:
    collect_exported: true
    ipaddress: '0.0.0.0'
    ports: '8080'
```
Add the `tomcat` stanza to `r_profile::lb::haproxy::listeners`

## Load balancer

/site/profile/lib/facter/source_ipaddress_targets.txt

```
192.168.13.110
```

* In order to resolve outgoing address to use to reach the load balancer, the load balancer IP Address needs to be present in `source_ipaddress_targets.txt`.  This file will be distributed by [pluginsync](https://docs.puppet.com/puppet/4.8/plugins_in_modules.html#auto-download-of-agent-side-plugins-pluginsync).  The easiest place for this file to live at is `./site/profile/lib/facter/source_ipaddress_targets.txt` in your control repo.  This file will be synced before each facter run, so it is not necessary to run puppet multiple times to setup the facts.
* If using the Vagrant lab, be sure to set `r_profile::web_service::tomcat::lb` to the IP address of your load balancer, NOT the hostname as the nodes do not have fully functional DNS
