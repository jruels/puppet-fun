# Selectors and Classes 
In this lab we are going to look at `selectors` and how we can use them to templatize our Puppet manifests.  With `selectors` we do not have to worry about difference between service names, package names or file locations.  We can define them at once time and then use variables throughout out manifests. 

We are also going to introduce `classes` which are a simple way to group selectors, variables and any other puppet code. 

## Selectors
### Puppet Master 
On the puppet master let's update our `nodes.pp` manifest with a selector specifying what the `ntp` service is called on `redhat` and `debian` distributions. 

Add the following to the top of the `nodes.pp` file. 
```
$ntpservice = $osfamily ? {
  'redhat'  => 'ntpd',
  'debian'  => 'ntp',
  default   => 'ntp',
} 
```

Now we need to update the `service` section of our manifest to use the new variable we defined.  

Update the manifest to look like the following for both our `wiki` and `wikitest` nodes. 
```
service { $ntpservice:
    ensure  => 'running',
    enable  => true,
  }
}
```

Save the file. 

### Apply manifest
Now let's log into the 	`Wiki`  and `Wikitest` servers, uninstall the `ntp` package and re-run the `puppet agent` to confirm the `selector` works as expected. 

After running the puppet agent confirm that `ntp` is running by using steps from previous labs. 

Did everything work?   

If you have questions or ran into a problem please reach out to the instructor for assistance. 

## Classes 
### Puppet Master 
The advantage of a Puppet class is that we can write our declaration once and not have to repeat it for every node.  We can group our nodes into classes and apply our manifests to any nodes inside of that class. 

Let's log into the Puppet Master and create a class at the very bottom of our `nodes.pp` manifest. 

In the `nodes.pp` we are going to create class `linux` which contains our selector and the code from our `wiki` node.
```
class linux {
  $ntpservice = $osfamily ? {
    'redhat'  => 'ntpd',
    'debian'  => 'ntp',
    default   => 'ntp',
  } 

  file { '/info.txt':
    ensure  => 'present',
    content => inline_template("Created by Puppet at <%= Time.now %>\n"),
  }
  
  package {'ntp':
    ensure  => 'installed',
  }

  service { $ntpservice:
    ensure  => 'running',
    enable  => true,
  }
}
```


Now we need to clean up all the code we moved into our class. 

Let's go ahead and remove all the content except for the class and the node definitions. 

Your file should look like this once complete. 
```
node 'wiki.example.com' {


}

node 'wikitest.example.com' {


}

class linux {

  $ntpservice = $osfamily ? {
    'redhat'  => 'ntpd',
    'debian'  => 'ntp',
    default   => 'ntp',
  }


  file { '/info.txt':
    ensure  => 'present',
    content => inline_template("Created by Puppet at <%= Time.now %>\n"),
  }

  package {'ntp':
    ensure  => 'installed',
  }

  service { $ntpservice:
    ensure  => 'running',
    enable  => true,
  }

}
```


Now we are going to specify that each node should use the `linux` class, by changing the node definition to include:
```
node 'wiki.example.lab' {

  class { 'linux': }

}

node 'wikitest.example.lab' {

  class { 'linux': }

}

```

Alright, the final file should now look like
```
node 'wiki.example.lab' {

  class { 'linux': }

}

node 'wikitest.example.lab' {

  class { 'linux': }

}

class linux {

  $ntpservice = $osfamily ? {
    'redhat'  => 'ntpd',
    'debian'  => 'ntp',
    default   => 'ntp',
  }


  file { '/info.txt':
    ensure  => 'present',
    content => inline_template("Created by Puppet at <%= Time.now %>\n"),
  }

  package {'ntp':
    ensure  => 'installed',
  }

  service { $ntpservice:
    ensure  => 'running',
    enable  => true,
  }

}
```

Now save this file and we are going to apply it to our Linux nodes. 

### Puppet nodes 
On each of the `Wiki` nodes login and run the `puppet agent` command. 

Did it work? 

Did you notice the `/info.txt` file was created on `Wikitest` since we hadn't created it previously? 

# Lab Complete 