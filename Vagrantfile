# -*- mode: ruby -*-
# vi: set ft=ruby :

domain = "demo.internal"
master = "puppet"
base_ip = "192.168.13."
pm_ip = base_ip + "10"
nodes = [
  "java-a.#{domain}", 
  "java-b.#{domain}", 
  "lamp-a.#{domain}", 
  "lamp-b.#{domain}",
  "wp-a.demo.internal",
  "wp-b.demo.internal",
  "lb.#{domain}", 
  "monitor.#{domain}"
]

def autostart(node)
  if node =~ /\-b/
    false
  else
    true
  end
end

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  # official centos image doesn't have vbox guest additions so shared
  # folders dont work.  installing them is a pain so use the puppet
  # one instead
  # config.vm.box="centos/7"
  config.vm.box="puppetlabs/centos-7.2-64-nocm"


  # common tasks for all hosts
  config.vm.provision "shell", inline: <<-SHELL
    # create an external fact so we can see we're running under vagrant
    mkdir -p /etc/facter/facts.d
    echo "vagrant=true" > /etc/facter/facts.d/vagrant.txt

    # install handy tools
    yum install -y wget links

    # restart network or our VM will be unreachable :(
    systemctl restart network

    # generate SSH keypair if we need to
    mkdir -p /vagrant/ssh_keys
    mkdir -p /root/.ssh
    chmod 700 /root/.ssh
    if [ ! -f /vagrant/ssh_keys/id_rsa ] ; then
      ssh-keygen -f /vagrant/ssh_keys/id_rsa -N ''
    fi
    # pick up existing generated key and use allow it to login.
    # we used to have a different key for each host but if you go
    # over 5 hosts the ssh-agent will get a failure on some hosts
    # because we run out of trys before we find the right key(!) so
    # just use the one keypair...
    cat /vagrant/ssh_keys/id_rsa.pub >> /root/.ssh/authorized_keys
    chmod 600 /root/.ssh/authorized_keys

    # for convenience, copy back the keypair to enable root to ssh to
    # localhost - this is good for things like testing out ssh+git
    cp /vagrant/ssh_keys/id_rsa* /root/.ssh/

    # firewall OFF (we will install iptables later...)
    systemctl disable firewalld
    systemctl stop firewalld

  SHELL


  config.vm.define :"#{master}.#{domain}" do |v|
    v.vm.network "private_network", ip: pm_ip
    v.vm.provider "virtualbox" do |vb|
      vb.memory = "6000"
      vb.cpus = "2"
    end
    v.vm.hostname = "#{master}.#{domain}" 
    v.vm.provision "shell", inline: <<-SHELL
      yum groupinstall -y 'Development Tools'
    SHELL
  end

  start_addr = 50
  (0..nodes.size-1).each do |i|
    node = nodes[i]
    config.vm.define node, autostart: autostart(node) do |v|
      vip = base_ip + (start_addr + (i * 10)).to_s
      v.vm.network "private_network", ip: vip
      v.vm.hostname = node
    end
  end


  # add puppmaster to every VMs hosts file
  config.vm.provision :hosts do |provisioner|
    provisioner.autoconfigure = true
    provisioner.sync_hosts = true
  end

end
