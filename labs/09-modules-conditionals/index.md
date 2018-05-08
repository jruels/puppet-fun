# Modules and Conditionals
In this lab we are going to create our own `mediawiki` module and install some of the required packages. We will also be dealing with classes and conditionals. 

## Puppet Master 
On the puppet master we need to enter the `modules` directory. 
```
cd /etc/puppetlabs/code/environments/production/modules
```

After we enter the directory it's time to create our own module.
```
sudo puppet module generate <yourname>-mediawiki --environment production
```

After we generate the module let's look at the directory structure and see what was created. 

Using `ls -l` we can see the following:
```
[vagrant@puppetmaster modules]$ ls -l mediawiki/
total 28
drwxr-xr-x. 2 root root 4096 May  3 00:36 examples
-rw-r--r--. 1 root root  492 May  3 00:36 Gemfile
drwxr-xr-x. 2 root root 4096 May  3 00:36 manifests
-rw-r--r--. 1 root root  342 May  3 00:36 metadata.json
-rw-r--r--. 1 root root  920 May  3 00:36 Rakefile
-rw-r--r--. 1 root root 3722 May  3 00:36 README.md
drwxr-xr-x. 3 root root 4096 May  3 00:36 spec
```

Let's look under the `manifests` directory 
```
[vagrant@puppetmaster modules]$ ls -l mediawiki/manifests/
total 4
-rw-r--r--. 1 root root 1060 May  3 00:36 init.pp
```

We can see there is an `init.pp` file which is mostly commented out but at the bottom it does contain a class for `mediawiki`
```
class mediawiki {


}
```

This is automatically loaded by our `nodes.pp` manifest and can be added directly to it just like how we are using the `linux` class. 

## PHP Selector 
As with `ntp` the packages for `PHP` are named differently on CentOS and Ubuntu. 

To fix this we need to create a selector to specify which package to install based on the OS. 

Add the following to our newly created `init.pp`
```
class mediawiki {
  $phpmysql = $osfamily ? {
    'redhat'  => 'php-mysql',
    'debian'  => 'php5-mysql',
    default   => 'php-mysql',
  }

  package { $phpmysql:
    ensure  => 'present',
  }

}
```

Now that we've added that to our `init.pp` we are going to update our `nodes.pp` manifest to use the new `mediawiki` class.  To do this just add the following under the `linux` class for both the `wiki` and `wikitest` nodes
```
class { 'mediawiki': }
```

After doing this save the file and we'll test it. 

## Puppet nodes
On both the `Wiki`  and `Wikitest` servers run the `puppet agent` and confirm it installed `php-mysql` as requested in the manifest.

To confirm it installed you can use `yum info` or `rpm -qa` on the `Wiki` server and `dpkg -l` or `apt-cache policy` on the `Wikitest` server. 

## Conditionals
A conditional allows us to specify logic around when Puppet does something.  In our example we are going to specify that a package is only installed on `Redhat` family OSes and ignored on all others. 

Update the `init.pp` to contain the following conditional under the `package` section. 
```
if $osfamily == 'redhat' {
  package { 'php-xml':
    ensure => 'present',
    }
  }
```
 

Now on the `Wiki` server run the Puppet agent again and see if it installs the package. 

After confirming it was installed on `Wiki` log into `Wikitest` and run the agent to confirm our conditional works correctly. 

## Puppet Forge modules
While it's fun to build out our own modules/classes, it's also great to use other people's work and save ourselves a little time and effort in the process.   We are going to use a couple modules off of the Puppet Forge to install the pre-requisites for `Mediawiki`. 

### Apache module 
We are going to start by installing the `Apache` module into our Puppet environment. This is a supported module which means it is updated and maintained by Puppet.  

To install it run the following on the Puppet server 
```
cd /etc/puppetlabs/code/environments/production/modules
sudo puppet module install puppetlabs-apache --version 3.1.0
```

You will now see it install the Apache module, as well as a few standard modules.
```
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-apache (v3.1.0)
  ├── puppetlabs-concat (v4.2.1)
  └── puppetlabs-stdlib (v4.25.1)
```

To list all of the installed modules we can run: 
```
sudo puppet module list
```

```
[vagrant@puppetmaster modules]$ sudo puppet module list
/etc/puppetlabs/code/environments/production/modules
├── js-mediawiki (v0.1.0)
├── puppetlabs-apache (v3.1.0)
├── puppetlabs-concat (v4.2.1)
└── puppetlabs-stdlib (v4.25.1)
/etc/puppetlabs/code/modules (no modules installed)
/opt/puppetlabs/puppet/modules (no modules installed)
```

Now let's learn a little more about how to use this module. 
Load the [Apache module page](https://forge.puppet.com/puppetlabs/apache#beginning-with-apache) in a browser and read the section about how to use it. 

As you can see the default settings will be used if we just add the class 
`class { 'apache': }` but this is not how we want to install it.  Our installation will include some settings specific for our environment.

In the `init.pp` we are going to add some code to our `mediawiki` class. 

At the bottom add the following to install and configure `Apache`
```
  class { '::apache':
    docroot    => '/var/www/html',
    mpm_module => 'prefork',
    subscribe  => Package[$phpmysql],
  }
```

This may be a little confusing so let's go through each of the different pieces of the code we added. 

`class { '::apache':` - This tells Puppet to install and enable the Apache service. The `::` tell Puppet to look in the top scope for the class, and not in our specific manifest or class. 
`docroot => '/var/www/html'` - Tells Apache where web files should live.
`mpm_module  => 'prefork'` - Tells Apache to enable a feature required for another package we are installing 
`subscribe  => Package[$phpmysql]` - Tells Puppet that `php-mysql` is required and to restart Apache if `php-mysql` is ever changed (updated, removed etc..)   

Now we need to enable `php` in our Apache module. 
To learn how to do this read about it on [Puppet Forge Apache page](https://forge.puppet.com/puppetlabs/apache#class-apachemodphp)

Go ahead and add the code required for enabling `mod::php` to our `init.pp` manifest under our `mediawiki` class. 

<details><summary>Click here for solution</summary>

<p>

<code>
class { '::apache::mod::php': }
</code>

</p>
</details>

After updating the `init.pp`  log into both the wiki servers and run the puppet agent to apply the new Apache class. 

## Confirmation 
Now confirm that Apache was installed and is running. 

### Wiki (CentOS)
```
sudo service httpd status
```

### Wikitest (Ubuntu)
```
sudo service apache2 status 
```

Both of these should come back showing the service is running.

Congratulations!  You just installed a module from Puppet Forge, created a couple new classes and successfully ran the puppet agent on a couple nodes to install some packages and configure them. 



## VCSrepo 
Now we need to clone the `Mediawiki` files from `GitHub` onto our Puppet nodes where Apache is already installed. Puppet does not have a built-in way to interact with Git repositories so we are going to install the `vscrepo` module the same way we installed the `Apache` module. 

In a browser go search for `vcsrepo` on the [Puppet Forge](https://forge.puppet.com)

Now on the Puppet master let's go ahead and install it using the command from the page
```
sudo puppet module install puppetlabs-vcsrepo
```

That command will download the latest version of `vcsrepo` from the Puppet Forge, which is what we want to do now, but it's always a good idea to choose a specific version when installing packages in production. 

To specify a version we can run: 
```
puppet module install puppetlabs-vcsrepo --version 2.3.0
```

Now when the puppet agent runs it will not automatically upgrade our `vcsrepo` module, but will require us to change the version before an upgrade can be performed.  This gives us time to test out new Puppet modules and see any bugs or incompatibilities before deploying to production. 

As with the `Apache` module we need to look at the [Config Page](https://forge.puppet.com/puppetlabs/vcsrepo#git) to learn how we can use this module. 

Add the code for doing a `clone` of `REL1_23` branch of the `mediawiki` repo to `init.pp` manifest under our `mediawiki` class. 

```  
  vcsrepo { '/var/www/html':
    ensure    => 'present',
    provider => 'git',
    source   => "https://github.com/wikimedia/mediawiki.git",
    revision  => 'REL1_23',
  }
```

Now run the puppet agent on both the `Wiki` and `Wikitest` servers.

Was it successful?   


On the `Wikitest` server you'll notice the Puppet agent threw an error that it couldn't clone the `Mediawiki` repo into `/var/www/html` because there was already a file in that directory. 

To confirm this run 
```
ls -l /var/www/html 
````

You can see that `index.html` is created on `Ubuntu` machines when `Apache` is installed. To resolve this issue we need to add a section that removes that file prior to cloning the `Mediawiki` repository. 

In `init.pp` add something like 

```
  file  { '/var/www/html/index.html':
    ensure   => 'absent',  
  }
```
Run the `puppet agent` on the `Wikitest` node and look at the output. 


Did it work? 

# Lab Complete 