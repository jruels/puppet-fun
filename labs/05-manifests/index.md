# Puppet Manifests

In this lab we are going to create some manifests to automate common tasks on our VMs using Puppet. 

## Node definitions
Start by creating simple node definitions in `/etc/puppetlabs/code/environments/production/manifests/nodes.pp` with the following. 

```
node 'wiki.example.lab' {

}

node 'wikitest.example.lab' {

}
```

Now let's add a section to create `/info.txt` and populate it with the date/time of last puppet run using the `inline_template` function. 
## Puppet manifest (create file)
Update `nodes.pp` to have the following. 
```
node 'wiki.example.lab' {

  file { '/info.txt':
    ensure  => 'present',
    content => inline_template("Created by Puppet at <%= Time.now %>\n"),
  }
}

node 'wikitest.example.lab' {

}
```

The `inline_template` function will return what we have in our quotations with the exception of an embedded ruby block (erb). An erb begins with `<%=`  contains any kind of Ruby expression, including variables and ends with `%>` 

Now that we've created our first manifest let's go see if it works. 

Log into the `Wiki` server 
```
vagrant ssh wiki 
```

Run the puppet agent 
```
sudo puppet agent --verbose --no-daemonize --onetime
```

If it was successful you will see output as below. 
```
Notice: /Stage[main]/Main/Node[wiki]/File[/info.txt]/ensure: defined content as '{md5}04fd1760f5f3ea5b0b754fcbf08e5198'
Notice: Applied catalog in 0.03 seconds
```
Ok great!  We can now see that Puppet created the file and populated it as requested. 

To confirm this we can `cat` that file 
```
cat /info.txt
```

Should see something like below:
```
Created by Puppet at 2018-05-02 05:20:28 +0000
```

## Client Filebucket 
Now that we've created this file, let's run the puppet agent again and see what happens. 
```
puppet agent --verbose --no-daemonize --onetime
```

What happened? Did it create a new file?  Did it update the old file? 

```
Notice: /Stage[main]/Main/Node[wiki]/File[/info.txt]/content: content changed '{md5}04fd1760f5f3ea5b0b754fcbf08e5198' to '{md5}4d4ed7c9996a8be770352265039fbb83'
Notice: Applied catalog in 0.04 seconds
```

We can see that the file was updated, so what would we do if we wanted to restore an older version of the file? 

Puppet has a feature called `Client Filebucket` which allows restoring of overwritten files.  To do this we need to know the `md5	` hash of the version we want to restore. 

To find this out we can look in `/var/log/messages` on the node itself. If we check for the last update applied to `/info.txt` we will see something like:
```
puppet-agent[29066]: (/Stage[main]/Main/Node[wiki]/File[/info.txt]/content) content changed '{md5}04fd1760f5f3ea5b0b754fcbf08e5198' to '{md5}4d4ed7c9996a8be770352265039fbb83'
```

Now that we have the `md5` hash let's go ahead and restore the file. 
```
sudo puppet filebucket -l --bucket /opt/puppetlabs/puppet/cache/clientbucket restore /info.txt 04fd1760f5f3ea5b0b754fcbf08e5198
```

Let's walk through this command a little. 

`-l` - tells the puppet agent the file we want to restore is on the local server. 
`--bucket` - path to the `clientbucket` directory 
`restore` - we want to restore the file 
`/info.txt` - file we want to restore 
`04fd1760f5f3ea5b0b754fcbf08e5198` - has we want to restore file to. 

To confirm this was successful go ahead and look at the `/info.txt` file. 
```
cat /info.txt
```

Look at the timestamp and compare to entries in `/var/log/messages` and you can see when the file was updated for the first time. 

What if you just want to see what was in the file at that hash?  To view the file and not actually restore it we can run the same command as before but change `restore` to `get` and remove the file path.
```
sudo puppet filebucket -l --bucket /opt/puppetlabs/puppet/cache/clientbucket get /info.txt 04fd1760f5f3ea5b0b754fcbf08e5198
```

Now it will print out the contents of the file for that hash. 
```
Created by Puppet at 2018-05-02 05:20:28 +0000
```

Backup a file to filebucket
```
sudo puppet filebucket backup /etc/group
```

Restore a file from filebucket to a different location 
```
sudo puppet filebucket restore /tmp/group <hash from above>
```

# Lab Complete 