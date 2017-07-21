# Setting up a load balancer (haproxy)
Luckily for you, Java, IIS and wordpress trails setup their own haproxy listers but you will still have to open the firewall port since we don't do this automatically:

```yaml
r_profile::lb::haproxy::open_firewall: true
```

Beyond this, you might still want to look at how things work to debug and explain them:

* The haproxy stats website will appear by default on your loadbalancer on port 9090, in the demo environment this can be reached at [http://lb.demo.internal:9090/](http://lb.demo.internal:9090/)
* The default username is `puppet` and the password is also `puppet`

## Change the stats username and password
```yaml
r_profile::lb::haproxy::stats_username: 'new_username'
r_profile::lb::haproxy::stats_password: 't0ps3cre4'
```

## Things to do
* Turn off one of the nodes being load balanced
* Put different content on the nodes being balanced
* Checkout the stats portal
* Checkout the access logs on the loadbalanced nodes

## Troubleshooting

Make sure the addresses listed in `/etc/haproxy/haproxy.cfg` are reachable.  If your using the vagrant lab, addresses starting `10.` are un-routable.  Make sure you have setup the source_ipaddress (adjust as needed):

/site/profile/lib/facter/source_ipaddress_targets.txt

```
192.168.13.110
````
