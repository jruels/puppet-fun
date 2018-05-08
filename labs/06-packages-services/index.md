# Packages and Services
Puppet can do many things that we will cover during this course however the 3 most basic are: 
* Package
* File
* Service 
Understanding how to interact with these 3 simple types will enable you to accomplish the majority of automation steps required to manage your infrastructure. 

## Packages 
To install a package is extremely easy!  All we need to do is define the package in our manifest file and apply it. 

### Puppet Master

On the Puppet master update `nodes.pp`  with the following to install the `ntp` package on both of our Wiki servers. 
```
node 'wiki' {

  file { '/info.txt':
    ensure  => 'present',
    content => inline_template("Created by Puppet at <%= Time.now %>\n"),
  }

  package {'ntp':
    ensure  => 'installed',
  }
}

node 'wikitest' {


  package {'ntp':
    ensure  => 'installed',
  }

}	
```

Notice we've added the `package` section to both `wiki` and `wikitest`. 

### Wiki (CentOS)
Now log into the `Wiki` server and confirm NTP is not already installed. 

```
sudo yum info ntp 
```

If it is already installed we need to remove it with 
```
sudo yum remove ntp 
```

Now let's go ahead and run the puppet agent on the `Wiki` node and confirm it installs NTP. 
```
sudo puppet agent --verbose --no-daemonize --onetime
```

Did it work?   Look for a line similar to 
```
/Stage[main]/Main/Node[wiki]/Package[ntp]/ensure: created
```

Also confirm NTP is installed by running 
```
sudo yum info ntp 
```

We should now see it as an `Installed Package`
```
...
Installed Packages
Name        : ntp
Arch        : x86_64
...
```

### Wikitest (Ubuntu)
Now let's log into the `Wikitest` server and confirm NTP is not installed yet. 
```
sudo dpkg -l |grep -i ntp 
```

If no results are returned it means that it is not installed. 

We can also check with `apt` 
```
apt-cache policy ntp 
```

Should show us something like 
```
ntp:
  Installed: (none)
```

If it is installed run the following to uninstall ntp 
```
sudo apt-get remove ntp 
```

Now if we do a `grep` we will see it's not running. 
```
ps auxww|grep -i [n]tp
```

We can also confirm it was completely removed with `apt-get`
```
apt-cache policy ntp 
```

It should say `Installed: (none)`

Great, now that we've confirmed it is not installed let's go ahead and use the same `puppet agent` command we just ran on the CentOS server to install it. 

Use `apt-cache policy` command from above to confirm `ntp` is now installed. 

Should return something like 
```
ntp:
  Installed: 1:4.2.8p4+dfsg-3ubuntu5.8
```

## Services 
Now that we've installed the `ntp` package we need to start it up, and ensure that is will start up even if the server is rebooted. 

### Puppet Master
To do this we once again log into our Puppet master and update our `nodes.pp` manifest to contain a `service` section like below. 
```
node 'wiki' {

  file { '/info.txt':
    ensure  => 'present',
    content => inline_template("Created by Puppet at <%= Time.now %>\n"),
  }

  package {'ntp':
    ensure  => 'installed',
  }

  service {'ntpd':
     ensure  => 'running',
     enable  => true,
  }
}

node 'wikitest' {


  package {'ntp':
    ensure  => 'installed',
  }

  service {'ntp':
     ensure  => 'running',
     enable  => true,
  }
}	
```

Look at what you just added and do you notice anything concerning? 

Ubuntu and CentOS have different names for the `ntp` service so we have to make sure we set `ntpd` for the CentOS VM and `ntp` for the Ubuntu VM.  We are going to discuss how to fix that a little later.  

Now that we've updated our Puppet master let's log into the Wiki nodes and confirm `ntp` is not running so that we can start it with Puppet. 

### Wiki (CentOS)
Log into the `Wiki` server and run the following to confirm the `ntp` service is not currently running. 
```
sudo service ntpd status 
```

If it is running go ahead and stop it with 
```
sudo service ntpd stop 
```

Then use `status` to confirm it was stopped. 

Now let's go ahead and run the `puppet agent` to apply the manifest.
```
sudo puppet agent --verbose --no-daemonize --onetime
```

Look at the output and if everything looks good go ahead and confirm it was started. 
```
sudo service ntpd status 
```

### Wikitest (Ubuntu)
Now log into the `Wikitest` VM and stop the `ntp` service. 
```
sudo service ntp stop 
```

After stopping it run the `puppet agent` and then use `service` to confirm it is running. 

```
● ntp.service - LSB: Start NTP daemon
   Loaded: loaded (/etc/init.d/ntp; bad; vendor preset: enabled)
   Active: active (running) since Wed 2018-05-02 18:05:11 UTC; 3min 34s ago
```

We can also confirm it is running by issuing:
```
ps auxww|grep -i [n]tp 
```

```
vagrant@wikitest:~$ ps auxww|grep -i [n]tp
ntp      16915  0.0  0.2 110032  5176 ?        Ssl  18:08   0:00 /usr/sbin/ntpd -p /var/run/ntpd.pid -g -u 112:116
```

Great! You have now installed the `ntp` package and started it on both CentOS and Ubuntu. 

## Uninstall package 
Now let's play around a little and see what the Puppet agent does if we uninstall `ntp` from both our Wiki servers. 

On the `Wikitest` server run the following to uninstall ntp 
```
sudo apt-get remove ntp 
```

Now if we do a `grep` we will see it's not running. 
```
ps auxww|grep -i [n]tp
```

We can also confirm it was completely removed with `apt-get`
```
apt-cache policy ntp 
```

It should say `Installed: (none)`

Ok, go ahead and run the `puppet agent` command from above and see what happens. 

Did it reinstall `ntp`?   
Did it start `ntp` as defined in the manifest? 

Run through the same process on the `Wiki` server using `yum`
```
sudo yum remove ntp
```

```
sudo service ntpd status 
```

```
yum info ntp
```

Run the `puppet agent` and confirm `ntp` was installed again and is now running. 

# Lab Complete