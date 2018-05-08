# Variables 

You've already used variables for creating our `ntpservice`  selector but we are going to use them a lot more in the following labs.  Variables allow us to define arrays and other options which will save us a lot of time in the future. 

Let's start by creating a variable to install a few packages. 

### Puppet Master
Log into the puppet master and add the following to the top of our `linux` class. 
```
  $admintools = ['git', 'lynx', 'screen']

  package { $admintools:
    ensure  => 'installed',
  }
```

This is going to run through the array and install each of the defined packages `git`, `lynx`, and `screen`. 

### Wiki 
Log into the `Wiki` server and confirm that these packages are not already installed. 
```
rpm -qa | egrep "^git|^lynx|^screen"
```

If nothing is returned then they are not already installed, however if you see a package that is installed already it must be removed with `yum -y remove <package>`. As an example let's say that `screen` was already installed so we would need to remove it with: 
```
sudo yum remove -y screen
```

Now if we run our `grep` command again it shouldn't return anything. 

Go ahead and run the `puppet agent` on the `Wiki` server and confirm it installs the requested packages. 
```
sudo puppet agent --verbose --no-daemonize --onetime
```

Now confirm the packages were installed either using `grep` command from above or by running them. 

Test `lynx` with the following command
```
lynx http://wttr.in
```
To quit just type `q` and hit `Enter`

Ok so it looks like our variable was successful! 

Now let's run the puppet agent on our `Wikitest` VM and confirm it works there as well. 

## Add additional packages 
There are lots of other things we can do with variables, since we already have a variable for `$admintools` let's go ahead and install some additional packages. 

Go update the `nodes.pp` on the Puppet master and update the array so that it installs 
* dos2unix
* asciidoc

Maybe you prefer `zsh` over `bash` for your shell.  Go ahead and update the manifest to install `zsh` on both the `wiki` and `wikitest` nodes.

Now on each of the nodes (`Wiki` & `Wikitest`) run the `puppet agent` command and confirm it installs the packages as requested. 

# Lab Complete 