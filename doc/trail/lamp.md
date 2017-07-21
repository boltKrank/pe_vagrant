# Setting up a LAMP server
LAMP is a handy name to refer to Linux systems running Apache, Mysql and PHP (or python... ;).  Setting up these four items is easy for experienced system administrators but can often result in inconsistent configurations if changes are made manually and by different operators.  Puppet Enterprise is a great way to setup such systems in an fast, efficient and consistent manner.

Separate profile classes has been created for each level of the stack to give maximum flexibility when deploying:
* `r_profile::database::mysql_server`
  * MySQL server and any required database instances
* `r_profile::web_service::apache`
  * Apache HTTPd server, PHP and database drivers
* `r_profile::webapp::git_site`
  * Deploys webapps from git to local directories
* `r_profile::webapp::php_config`
  * Writes PHP configuration files with specified settings, eg, to ensure that PHP can connect to a database

Each aspect of the stack can be configured through hiera:

## Database

```yaml
r_profile::database::mysql_server::db:
  quote:
    user: 'quote'
    password: 'quote'
    host: 'localhost'
    sql: '/var/www/html/res/sql/schema.sql'    
    grant:
      - 'CREATE'
      - 'SELECT'
      - 'UPDATE'
      - 'INSERT'
      - 'DELETE'
```
## Load balancer and apache

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

## Git site

```yaml
r_profile::webapp::git_site::sites:
  '/var/www/html':
    source: 'https://github.com/GeoffWilliams/phpquote'
```

## PHP settings

```yaml
r_profile::webapp::php_config:
  '/var/www/html/res/config/config.php':
    vars:
      db_user: 'quote'
      db_password: 'quote'
      db_host: 'localhost'
      db_name: 'quote'
```

## Testing
Once the above hiera data has been deployed, you can try out the application you installed by browsing to the application you installed by browsing to:

[http://lamp-a.demo.internal/quote/1](http://lamp-a.demo.internal/quote/1)

## Monitoring the database
Open the firewall:

```yaml  
r_profile::database::mysql_server::enable_firewall: true
```

## Things to do

* Browse locally (`links http://localhost/quote/1')
* Browse via the load balancer at [http://lb.demo.internal/quote/1](http://lb.demo.internal/quote/1) and try turning things on and off
* Try adapting for different PHP applications

## Debugging

* Check the PHP error logs at `/var/log/httpd/default-site_error.log`
* Check you used the same database connection details at each point in your hiera data
* Check the database parameters are correct at `/var/www/html/res/config/config.php`
* Check the database works and accepts connections
