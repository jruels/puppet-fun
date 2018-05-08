# Additional Modules 

Before we get into the next lab let's finish up our previous lab.  The last puppet agent run did not do what we wanted, which was to remove `/var/www/html/index.html` prior to cloning the `Mediawiki` repository.  This is because Puppet does not always run in the order we need so we have the option of specifying exact run order. 

Update `init.pp` so that it has the following right below our file resource for `/var/www/html/index.html`
```
File['/var/www/html/index.html'] -> Vcsrepo['/var/www/html']
```

This tells Puppet to remove `index.html` prior to cloning the `Mediawiki` repo. 

Log into `wikitest` and run the puppet agent.  Confirm it successfully deletes `/var/www/html/index.html` and clones our `Mediawiki` repository. 

Now that's complete let's continue this lab by installing and configuring some additional modules required for installing MediaWiki. 

## MySQL 
Visit the [Puppet Forge](https://forge.puppet.com) and search for `mysql`. Once you find the supported MySQL module install it using steps from previous lab.

After it installs go back to the [MySQL module page](https://forge.puppet.com/puppetlabs/mysql#setup) and read through how to get started with this module. 

Now using what you learned on the MySQL module page update 	`init.pp` to install and configure a MySQL server with a root password of `training`


Run the puppet agent on both `Wiki` and `Wikitest` to install MySQL and configure it as specified in `init.pp`


### Confirmation 
Now to confirm `MySQL` was installed and configured correctly run the following on both `Wiki` and `Wikitest`
```
mysql -u root -ptraining
```

If this works you should now be logged into MySQL and sitting a prompt like this:
```
mysql>
```

To quit we can type `\q`

Great! MySQL is now installed on both servers. 

## Firewall module
Now that we've got Apache, MySQL, and PHP installed and configured it's time to configure Puppet to manage our IPTables firewall. 

Go to the [Puppet Forge](https://forge.puppet.com) ,search for the `firewall` module, and select the one that shows `Supported`

To  install it on the Puppet master run:
```
sudo puppet module install puppetlabs-firewall --version 1.8.0
```

You'll notice that the latest version is 1.12.0, but there's a bug in that version where it forces you to use `ipv6` and this causes it to fail on our VMs. 

After it is installed use the documentation on the [Puppet Forge Firewall](https://forge.puppet.com/puppetlabs/firewall) page to see how to enable it in  `init.pp` . 


```
  class { '::firewall': }
  
  firewall { '000 allow http access':
    port    => '80',
    proto  => 'tcp',
    action => 'accept'
  }  

  firewall { '001 allow SSH access':
    port    => '22',
    proto  => 'tcp',
    action => 'accept'
  }  

```
## Maybe Optional
Before we can run Puppet on the `Wiki` we have to create an IPTables file.

### Wiki (CentOS)
Let's start by looking at the IPTables configuration file. 
```
sudo vi /etc/sysconfig/iptables 
```

You should notice that it is blank, this is because CentOS 6 by default does not have any firewall rules enabled. 

To add a rule for Puppet master (TCP 8140)  add the following to iptables config file. 

```
*filter
:INPUT ACCEPT [418:23072]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [213:15108]
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8140 -j ACCEPT
COMMIT
```

Now you can see that we have a rule stating that TCP connections on port 8140 are accepted.

To enable this we can run 
```
sudo service iptables restart 
```


Now run the puppet agent on both `Wiki` and `Wikitest` and confirm it added the rules specified in our manifest to the firewall. 

```
sudo iptables -L |egrep "000|001"
```

If you see the FW rules you expected then everything is good.  If not try to troubleshoot and if issue is not resolved ask the instructor for assistance. 





