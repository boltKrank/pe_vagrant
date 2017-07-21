# PEPOC
_A Puppet Enterprise Proof-of-concept kit_

## How it works
PEPOC uses [Puppetizer](https://github.com/GeoffWilliams/puppetizer) to automatically install Puppet Enterprise masters and agents as described in the [inventory file](inventory/hosts).

PEPOC comes with a virtual lab of VMs for your convenience but it can also be configured to use real VMs on public or private cloud.  The idea is that the workstation your running the puppetizer tool from reaches out over the network using SSH and up puppet according to your requirements.

## What's in the box?

* Vagrant lab
* Inventory file describing Vagrant lab
* Gemfile to load correct versions of puppetizer, etc.
* Instructions

## Setup
In a nutshell:

1. Setup your workstation
2. Setup your POC VMs
  * Built-in Vagrant lab
  * _Real_ VMs
3. Use Puppetizer to install masters and agents
4. Customization workflows

### 1. Setup your workstation

1. Install open-ssh client, ssh-agent and md5sum (`brew install openssl md5sha1sum`)
2. Install ruby
3. `sudo gem install bundler`
4. `git clone https://github.com/puppetlabs/pepoc.git`
5. `cd pepoc`
6. `bundle install`
  * You might need to install additional OS libraries to get this operation to succeed

#### 1.1 Optional steps - if using Vagrant

1. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. Install [Vagrant](https://www.vagrantup.com/downloads.html)
3. `vagrant plugin install vagrant-hosts`
4. `vagrant plugin install vagrant-hostsupdater`

NOTE
----
Mac users will need to install xcode (from app store) and run `xcode-select --install`

##### 1.1.1 Vagrant lab host entries (recommended)
To create `/etc/hosts` entries for the vagrant machines in the lab on the workstation you are running from, install the `vagrant-hostsupdater` plugin:

To avoid being prompted for a password when updating `/etc/hosts` follow the plugin's
[instructions](https://github.com/cogitatio/vagrant-hostsupdater#passwordless-sudo).

### 2. Setup your POC VMs
#### 2.1 Using the built-in Vagrant lab
```shell
vagrant up
```

#### 2.2 Using _real_ VMs
VMs Must:

* Be RHEL/CentOS (Windows agents must be setup manually)
* Accept incoming SSH connections from the machine your running the PEPOC from
* Allow one of the following:
  * Login as root via SSH using a password
  * Login as root via SSH using a public key
  * Login as a regular user via SSH and obtain root via passwordless sudo
  * Login as a regular user via SSH and obtain root via sudo + password
  * Login as a regular user via SSH and obtain root via su + root password

#### 2.3 Setup SSH authentication
By far the easiest and securest way to run the POC is to load keys into the SSH agent and not worry about things:

##### 2.3.1 Public Key based authentication
First start the agent
---------------------
```shell
eval `ssh-agent -s`
```

Now add your key
-----------------
```shell
ssh-add ssh_keys/id_rsa
```
The Vagrant lab dumps keys into the above directory.  If your using _real_ VMs, you will need to obtain a copy of the private key for the account your trying to access and then run the `ssh-add` against it.  Make sure your keys have permission `0400` or they will not be loaded.

If you don't yet have key based authentication in place, then its easiest to generate a new keypair locally and use ssh-copy-id to install it.  Mac users will need to `brew install ssh-copy-id` to obtain the command.  See https://valdhaus.co/writings/ansible-post-install/ for a worked example.

NOTE
----
Your vagrant keys will be regenerated when you reprovision your lab VMs.  If you destroy or recreate your key you must remove the old one from the SSH agent or you will not be able to access lab machines after creating a new one:
```shell
ssh-add -D #remove all keys
```

##### 2.3.2 Password based authentication
To stop passwords appearing in the process table, they are passed by exporting the variable `PUPPETIZER_USER_PASSWORD`, e.g.:

```shell
export PUPPETIZER_USER_PASSWORD=t0ps3cret
```

NOTE
----
You may encounter the following error if you have SSH keys loaded in the SSH agent:
```shell
disconnected: Too many authentication failures for root (2) @ #<Net::SSH::Simple::Result exception=#<Net::SSH::Disconnect: disconnected: Too many authentication failures for root (2)> finish_at=2016-09-25 23:54:48 +1000 stderr="" stdout="" success=false>
```

In this case, the fix is to unload all loaded keys:

```shell
ssh-add -D
```

#### 2.4 Inventory file
Setup your [inventory file](/hosts/inventory).  If you are using the Vagrant lab, then this is already done for you although you may want to tweak it.  If you are using _real_ VMs, then you will need to adjust as follows:

* Under the `[puppetmasters]` heading, list the address(es) of hosts to install as monolithic masters
* Under the `[agents]` heading, list the address(es) of hosts to install as agents
* For each node, specify `pp_role` if you would like to assign a role class via CSR attributes
* For your master, set `deploy_code=true` to checkout an R10K control repo
* For the moment, this file has to be edited in-place.  You might want to fork the pepoc repository to service a new client.  Please don't commit back customer specific changes to the main repo

### 3. Use Puppetizer to install masters and agents

#### 3.1 Getting help
```shell
bundle exec puppetizer --help
```

#### 3.2 Common options and techniques
##### SSH username

Puppetizer attempts to login as root by default, if you need to login as another user, pass the username like this:

```shell
--ssh-username username
```

##### Debugging/Stack traces
To get stack traces, invoke puppetizer like this:

```shell
puppetizer --verbosity debug
```

##### Sudo
If your username is not `root`, puppetizer will attempt to use `sudo` to gain root access.  If you need to type a password when using sudo, set the `PUPPETIZER_USER_PASSWORD` shell variable to the password you need to type, eg:

```shell
export PUPPETIZER_USER_PASSWORD=t0ps3cret
```

##### Running
Be sure to always launch puppetizer using bundler or the wrong libraries may be used:

```shell
bundle exec puppetizer ...
```

##### Different usernames/passwords for different hosts
Puppetizer assumes the same username and password is used for all machines in the inventory file.  This is to avoid building a plaintext file containing all the usernames and passwords to a fleet of machines...

If you find yourself in the situation where there are lots of different usernames and passwords needed, your options are:

* Iteratively set `PUPPETIZER_USER_PASSWORD` and comment out hosts in the inventory file
* Use `ssh-copy-id` to manually login _once_ to the required hosts and then use public key authentication
* Provide passwords in a [CSV file](https://github.com/GeoffWilliams/puppetizer#password-file)
* Think of something better and create a PR :)

#### 3.3 Prepare the PE media
You must manually download the PE media from puppet.com and place the compressed tarball in the pepoc directory.  This is to ensure that marketing leads are captured.  You will need the 64 bit RHEL 7 version.

IMPORTANT:  You must use 2016.4 LTR release of Puppet Enterprise.  Quarterly releases are not supported and will result in failed installation due to changes to built-in node classifier rules.

#### 3.4 Install the puppet master(s)

```
bundle exec puppetizer puppetmasters
```

This will:

* Process any CSR role definitions
* Install PE on the master
* Clone a bare repository from https://github.com/GeoffWilliams/r10k-control/ to `/var/lib/psquared/r10k-control` - this becomes the POC's git server
* Configure the master with a nice shell prompt for root, all known agents, etc.

#### 3.5 Install the puppet agents

```
bundle exec puppetizer agents
```

This will:

* Process any CSR role definitions
* Attempt to install the puppet agent from the master on each agent listed in the inventory file

##### NOTE
* If installation fails on a particular host, puppetizer will continue to the next host

#### 3.6 Summary
By this stage, you should have a functional puppetmaster and agents and be ready to continue.  Its also possible to run:

```shell
bundle exec puppetizer all
```

### 4 Customization workflows

The steps below outline the common procedure that each of the [customisation trails](doc/trail/) follow.

`puppet.demo.internal` is the hostname to use for the demo environment, update this to suit your needs if your not using the Vagrant lab:

1. Checkout a copy of the control repo from the puppet master
  * On the master:
    * `git clone /var/lib/psquared/r10k-control`
  * On a workstation:
    * Obtain private key from the master at `/var/lib/psquared/.ssh/psquared@puppet.demo.internal`, download to your workstation and `chmod 400` it.
    * Add the key to the SSH agent: `ssh-add psquared@puppet.demo.internal`
    * Clone the repository somewhere: `git clone ssh://psquared@puppet.demo.internal/var/lib/psquared/r10k-control/`

2. Add your local changes inside the cloned r10k-control directory:
  * The bulk of changes involve customising the ready profiles through their parameters by adding hiera data to `/hieradata`
  * If you decide to go off-trail and make new roles and profiles, put them in the `role` and `profile` directories under `/site`
  * Reference new modules in `/Puppetfile`
3. git commit... git push - code manager will be triggered to update the code on master on push.  The git command will freeze until Code Manager has finished running so that you know your code is live when you want to run it.  
4. run puppet on affected nodes as needed or wait 30 minutes

#### 4.1 To-go customization trails
The following customizations are _ready to go_ so feel free to attempt them:

* [Setting up a load balanced GeoServer](doc/trail/geoserver.md)
* [Setting up a monitor server](doc/trail/monitor.md)
* [Setting up a load balancer (haproxy)](doc/trail/lb.md)
* [Setting up A LAMP server](doc/trail/lamp.md)
* [Setting up A LAMP server with WordPress](doc/trail/wordpress.md)
* [Setting up IIS on Windows](doc/trail/iis.md)
* [Setting up the iptables firewall on Linux](doc/trail/iptables.md)

# Vagrant cheat sheet

### Start all VMs

```shell
vagrant up
```

### Pause all VMs

```shell
vagrant suspend
```

### Destroy all VMs

```shell
vagrant destroy -f
```


### SSH to a VM
`vagrant ssh HOSTNAME`

E.G.,
```shell
vagrant ssh puppet.demo.internal
```

# FAQ
Q:  What IP addresses do the Vagrant lab VMs use?

A:  Check the value of `base_ip` in the [Vagrantfile](Vagrantfile) to determine the subnet.  The puppetmaster uses the `.10` address and all agent nodes are allocated sequentially from `.50` in increments of 10.  Hopefully you won't have to worry about the IP addresses too much since the vagrant-hostsupdater will add them the /etc/hosts file on your workstation.

Q: I receive the error: "An internal Escort error has occurred, you should probably report it by creating an issue on github! To get a stacktrace, up the verbosity level to DEBUG and execute your command again"

A: The root-cause of the error should be in the line above this one.  Errors are not currently handled properly as this code is still experimental.

Q: I can't SSH!  I might have added some VMs to Vagrantfile too...

A: I ended up having to reboot the laptop and then `vagrant destroy -f` to fix this.  Something real skrewy was going on around memory, firewalls and duplicate IPs...
