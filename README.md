# Puppet Fundamentals 


## Testing collapsible code. 
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

