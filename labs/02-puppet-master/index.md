# Install Puppet Master 

The following steps will be performed on the Puppet Master VM we created in the last lab. 

## Start up Puppetmaster VM 
```
cd /path/to/puppet-fun
vagrant up puppetmaster
```

## Log into VM 
To log into the Puppet Master using Vagrant type the following in a shell.
```
vagrant ssh puppetmaster
```

You should now be logged into the VM and can start installing the Puppet Master. 

## Install Puppet Server
First we need to install some pre-requisite packages and add the `YUM` Puppet repository to the VM, by running the following command. 
```
sudo yum install -y https://yum.puppetlabs.com/puppet5/puppet-release-el-6.noarch.rpm git ntp
```

Now we need to start and enable `ntpd`
```
sudo service ntpd start && sudo chkconfig ntpd on
```

To confirm `ntpd` started successfully and is enabled to start at boot we can run the following commands. 

```
sudo service ntpd status 
sudo chkconfig --list |grep ntpd 
```

You should see something like the following. 
```
ntpd (pid  1563) is running...
```
```
ntpd           	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```


After installing the dependencies and adding the Puppet repository we can now install the Puppet server. 
```
sudo yum -y install puppetserver 
```

By default the Puppet Server will use 2G of memory and since we don't require that let's go ahead and change it to 1G 

Open `/etc/sysconfig/puppetserver` in a text editor and change `Xms2g` and `Xmx2g` to `Xms1g` and `Xmx1g`

```
sudo vim /etc/sysconfig/puppetserver
```

```
JAVA_ARGS="-Xms2g -Xmx2g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

So it ends up looking like:
```
JAVA_ARGS="-Xms1g -Xmx1g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

Now save the file and we will start the service. 
```
sudo service puppetserver start 
sudo chkconfig puppetserver on
```

Now add the Puppet binaries to the local `PATH` so they are easier to access. 
```
echo "PATH=/opt/puppetlabs/bin/:$PATH" >> ~/.bash_profile
```

Source  `~/.bash_profile` to update environment with new `PATH`
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

## Confirmation 
Confirm Puppet was installed successfully. 

First check the Puppet Server 
```
puppetserver --version 
```
Should return something like
```
puppetserver version: 5.3.1
```

Now confirm the agent was also installed 
```
puppet --version 
```

And you should see 
```
[vagrant@puppetmaster ~]$ puppet --version
5.5.1
```

# Lab Complete 