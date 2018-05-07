# Puppet Agent


Now we are going to spin up our Wiki servers.  Remember Wiki is CentOS 6 and Wikitest is Ubuntu.  This will allow us to see how Puppet can work with different distributions.   

## Setup Wiki VMs
Now that we've got the Puppet server all setup and ready to go we need to add some VMs that we can manage with Puppet.

Startup the Wiki and Wikitest VMs with Vagrant
```
vagrant up wiki wikitest 
```

These will take a little while to download the boxes and boot up. 

Once the installation is complete we are going to configure the `/etc/hosts` file so that they know where the Puppet master is and we don't need to worry about DNS. 

Log into `Wiki` and `Wikitest`
```
vagrant ssh <wiki> or <wikitest>
```

Now become the root user
```
sudo -i 
```

Use `echo` to add files to `/etc/hosts`
```
echo 172.31.0.201 puppetmaster.example.lab puppet >> /etc/hosts
```

Test to confirm the VM is responding to `puppetmaster` 
```
ping -c 1 puppetmaster.example.lab 
```

and you should see a response 

```
PING puppetmaster (172.31.0.201) 56(84) bytes of data.
64 bytes from puppetmaster (172.31.0.201): icmp_seq=1 ttl=64 time=0.262 ms
```

## Install Puppet agent 

### CentOS 6
Let's start by installing the agent on our Wiki VM, which is a CentOS machine. 

First add the Puppet repository like we did on the master. 
```
sudo yum install -y https://yum.puppetlabs.com/puppet5/puppet-release-el-6.noarch.rpm
```

Now go ahead and install the Puppet agent. 
```
sudo yum install -y puppet-agent
```

After we install the Puppet agent it's time to update the `puppet.conf` file with the Puppet server name. 

```
sudo vim /etc/puppetlabs/puppet/puppet.conf
```

Now add the following to that file. 
```
[agent]
    server = puppetmaster.example.lab
```

Add Puppet binaries to `PATH`
```
echo "PATH=/opt/puppetlabs/bin/:$PATH" >> ~/.bash_profile
```

Source
```
source ~/.bash_profile
```

Add binaries to `sudo` path as well. 
```
sudo visudo 
```

In the file look for the line 
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
```

Now add `:/opt/puppetlabs/bin` to the end so it looks like:
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/opt/puppetlabs/bin
```

Now we are going to start the Puppet agent and set it to run on VM reboot. 
```
sudo puppet resource service puppet ensure=running enable=true
```

This will also send a certificate request to the master so that we can manage the VM using Puppet. 

### Ubuntu 
Now that we've installed the Puppet agent on CentOS 6 let's do the same on our Wikitest Ubuntu server. 

Download the Puppet repo file. 
```
wget https://apt.puppetlabs.com/puppet5-release-trusty.deb
```

Now install it 
```
sudo dpkg -i puppet5-release-trusty.deb
```

After it's installed let's update the package repo. 
```
sudo apt update -y 
```

Now we can install the agent package
```
sudo apt install puppet-agent -y
```

Now just like on CentOS we need to set the `server` in the `puppet.conf` file. 

```
sudo vim /etc/puppetlabs/puppet/puppet.conf
```

Now add the following to that file. 
```
[agent]
    server = puppetmaster.example.lab
```

Add Puppet binaries to `PATH`
```
echo "PATH=/opt/puppetlabs/bin/:$PATH" >> ~/.bash_profile
```

Source
```
source ~/.bash_profile
```

Add binaries to `sudo` path as well. 
```
sudo visudo 
```

In the file look for the line 
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
```

Now add `:/opt/puppetlabs/bin` to the end so it looks like:
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/opt/puppetlabs/bin
```

Now let's start the Puppet service and enable it to start on boot. 
```
sudo service puppet start
sudo update-rc.d puppet enable
```

Now generate the cert request
```
sudo puppet agent --verbose --no-daemonize --onetime
```

## Puppet certificate requests
Log into the `puppetmaster` to sign the certs from agents. 
```
vagrant ssh puppetmaster
```

List out requests 
```
sudo puppet cert list 
```

You should see requests from both Wiki and Wikitest 
```
[vagrant@puppetmaster ~]$ sudo puppet cert list
  "wiki.fios-router.home"     (SHA256) 25:EF:70:71:A8:1A:B1:39:90:33:BB:A4:E5:1E:08:F8:95:5C:87:BA:4D:4B:59:96:E8:28:36:C7:70:A0:B1:05
  "wikitest.fios-router.home" (SHA256) 2E:C4:F3:72:45:E8:E3:41:F4:B9:03:7F:0E:3D:65:32:0D:2A:EA:BD:BF:4E:EC:45:C0:FA:9D:EF:C8:5D:37:87
```

Now to sign the requests. 

```
sudo puppet cert sign wiki.example.lab 
```

We can also sign all of the requests at once 
```
sudo puppet cert sign --all 
```

## Master/Agent connection 
Now that we've signed the certificates let's make sure the agents can communicate with the master. 

## Wiki (CentOS) 
Log into the `wiki` server 
```
vagrant ssh wiki 
```

Test agent connection 
```
sudo puppet agent --test
```

If everything is working correctly you should see 
```
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for wiki.example.lab
Info: Applying configuration version '1525234880'
Notice: Applied catalog in 0.01 seconds
```

### Wikitest (Ubuntu)
Now confirm it works on the `wikitest` server as well 
Log into the `wikitest` server
```
vagrant ssh wikitest 
```

Test agent connection 
```
sudo puppet agent --test 
```

You should see this: 
```
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for wikitest.example.lab
Info: Applying configuration version '1525234880'
Notice: Applied catalog in 0.01 seconds
```

# Lab Complete 