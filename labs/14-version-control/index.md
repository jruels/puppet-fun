# Version Control
In this lab we are going to setup the `puppet agent` to start on boot and to also run every 60 seconds. 



## Start Puppet service
So far we can been running the `puppet agent` manually but in a production environment we want it to run all the time so it will automatically apply any changes that we make on the Puppet master. 

## Wiki 
On the `wiki` server we need to run two commands to enable Puppet on boot as well as to start the puppet service. 

Start Puppet service 
```
sudo service puppet start
```

Enable agent on boot
```
sudo chkconfig puppet on 
```

## Wikitest 
On `wikitest` we are going to do the same thing but the command to enable on boot is a little different 

Start Puppet service 
```
sudo service puppet start
```

Enable agent on boot
```
sudo update-rc.d puppet enable 
```

Once this is complete let's make the following change to both `wiki` and `wikitest`

Update the `/etc/puppetlabs/puppet/puppet.conf` file and add the `runinterval`

```
sudo vim /etc/puppetlabs/puppet/puppet.conf
```

Add the following:
```
[main]
  runinterval = 60
```

Now that we've added that we need to restart the puppet agent to apply the settings.

On both `wiki` and `wikitest` run
```
sudo service puppet restart
```

Now the agent will check in with the master ever 60 seconds. 

## Version control 
As we've gone through the last few days and continually updated our configuration files, manifests and modules there have probably been times where you wished it was possible to "undo" your last change to a file.  Well we are going to setup `Git` which is a version tracking system and will allow us to take snapshots of our configuration files and revert to a working copy at any time.

Start out by setting some global variables. 
```
sudo git config --global user.name "<Your Name>"
sudo git config --global user.email "<Your Email>"
```

The above values are used to track who actually made changes to a file.  In a shared environment it is critical to have a log of all the changes made and who committed them. 

Now that we've configured those settings let's `cd` into our Puppet directory and initialize our `Git` repository. 
```
cd /etc/puppetlabs/code/environments/production
sudo git init 
```

Add all of the files to `Git`
```
sudo git add . 
```

We need to commit all of these files for the first time so let's go ahead and do that. 
```
sudo git commit -m "initial commit"
```

All of our files are being tracked in `Git` and if we make a mistake we can rollback very easily now. 

To confirm this is working and see all the commits to our repository we can run 
```
sudo git log 
```

This shows us the following: 
* commit hash
* author (You) 
* commit message

## Git file recovery 
Now that we've got `Git` setup let's go ahead and run through a scenario where we "accidentally" change a config file and break something, and then use `Git` to restore a working copy. 

Let's say that you are trying to clean up the `LocalSettings.erb` file and so you start deleting some of the commented lines. 

open up the file and delete some random lines, make sure you get some of the required `$wg` variables to actually break things. 
```
sudo vim /etc/puppetlabs/code/environments/production/modules/mediawiki/templates/LocalSettings.erb
```

Delete some of the 

After "cleaning" up the file let's go ahead and commit it with `Git`

```
sudo git add . 
sudo git commit -m "cleanup" 
```

Now if we wait for the `puppet agent` to run, and go load our `Mediawiki` site we will notice an error saying it can't connect to the database. 

To confirm what the issue is we can run 
```
sudo git log 
```

This shows us the last commit, which is what broke the `wiki` and `wikitest` servers.

To fix this we can `revert` to a working version. 

```
sudo git revert --no-edit <commit that broke it>
```

Let's explain what this command actually does. 

`--no-edit` - by default `Git` will open our systems default editor to create a new message around why we are reverting. This tells it not to do that. 
`<commit>` - This is the change that we just committed, and it will `revert` that commit and our files will be just like before that was committed. 

After running above command let's look at the log again.
```
sudo git log 
```

Now we will see at the top of the log it reverted our previous commit. 

Let's confirm that it did in fact fix everything. 
```
sudo vim /etc/puppetlabs/code/environments/production/modules/mediawiki/templates/LocalSettings.erb
```

Now scroll down until you see where you deleted the `wg` database variables. 

Were they restored? 

Now go load up both Mediawiki sites and confirm they work again. 

Sometimes we make changes to a file and don't have a chance to actually commit the file.  In this case it wouldn't really make sense to revert to the last commit so instead we can use the `git reset` command. 

On the Puppet master edit your `LocalSettings.erb` and delete a bunch of lines from it, making sure to delete some of the database variables. 

Now if you save that file and wait a minute you'll notice both mediawiki sites are down.  

To resolve this we can reset the git repository to before we made those changes by running 
```
sudo git reset HEAD --hard 
```

To confirm this worked go ahead and reload your mediawiki websites and they should be back online. 

You can also look at the `LocalSettings.erb` file and confirm it has all the lines you previously deleted. 

# Lab Complete 



