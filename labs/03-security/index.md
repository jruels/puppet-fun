# Puppet Server Security

Because Puppet uses SSL to connect to agents it requires some preliminary configuration steps before we can actually use it to apply configuration to other nodes. 

## SELinux 
SELinux is a security feature that blocks access to services and directories/files if they are owned by a user or group that is not known to the system. 

To avoid issue with SELinux during our lab exercises we are going to set it to `Permissive` so that it will keep a record of things that would have been blocked but not actually block anything. 

To confirm the mode SELinux is running in we can execute the following command. 
```
getenforce 
```

This should show either `Enforcing`, or  `Permissive` 

```
[vagrant@puppetmaster ~]$ getenforce
Enforcing
```
To set SELinux to permissive mode is very easy, just log into your VM and run the following command 
```
sudo setenforce permissive 
```

You should get the following confirming that it was updated. 

```
[vagrant@puppetmaster ~]$ getenforce
Permissive
```

Now we need to make this change permanent across reboots by running the following `sed` command to update the `SElinux` configuration file. 
```
sudo sed -i 's/=enforcing/=permissive/g' /etc/sysconfig/selinux
```

Now confirm it was updated successfully by running 
`grep SELINUX= /etc/sysconfig/selinux` and you should see:

```
# SELINUX= can take one of these three values:
SELINUX=permissive
```

Looks like our command was successful and SELinux will stay in permissive mode even across reboots. 

## SSL certificates 
SSL certificates are used to secure connections between the Master and any nodes.  They are also used as a way to make sure that an unauthorized node does not gain access to your master without your approval.  

When we started the Puppet server it automatically generated SSL certs for us and placed them in the SSL directory specified in `puppet.conf`. To confirm these were created run the following. 
```
sudo ls -l /etc/puppetlabs/puppet/ssl
```
You should see something like 
```
total 28
drwxr-xr-x. 2 puppet puppet 4096 May  1 17:59 certificate_requests
drwxr-x---. 2 puppet puppet 4096 May  1 17:59 private
drwxr-xr-x. 2 puppet puppet 4096 May  1 17:59 public_keys
drwxr-x---. 2 puppet puppet 4096 May  1 17:59 private_keys
drwxr-xr-x. 2 puppet puppet 4096 May  1 17:59 certs
drwxr-xr-x. 5 puppet puppet 4096 May  1 21:49 ca
-rw-r--r--. 1 puppet puppet  983 May  1 21:49 crl.pem
```

Under each of the directories we can also see that certificates and keys were generated 
```
sudo ls -l /etc/puppetlabs/puppet/ssl/certs 
```

```
total 8
-rw-r--r--. 1 puppet puppet 2009 May  1 17:59 ca.pem
-rw-r--r--. 1 puppet puppet 2053 May  1 17:59 puppetmaster.fios-router.home.pem
```

```
sudo ls -l /etc/puppetlabs/puppet/ssl/private_keys/
```

```
total 4
-rw-r-----. 1 puppet puppet 3243 May  1 17:59 puppetmaster.fios-router.home.pem
```

Now that we've confirmed the certs and keys were generated let's continue on with opening the firewall ports. 

## Firewall 
To allow agents to connect to the Puppet server we must confirm tcp port 8140 is open to all nodes. 

Let's start by looking at the IPTables configuration file. 
```
sudo vi /etc/sysconfig/iptables 
```

You should notice that it is blank!   This is because CentOS 6 by default does not have any firewall rules enabled. 

To add a rule for Puppet master (TCP 8140) and a rule for SSH (TCP 22) we could add the following to our iptables config file. 

```
*filter
:INPUT ACCEPT [418:23072]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [213:15108]
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8140 -j ACCEPT
COMMIT
```

Now you can see that we have a rule stating that TCP connections on port 8140 are accepted and TCP connections on port 22 are accepted. 

To enable this we can run 
```
sudo service iptables restart 
```

Now in our lab environment we have not blocked any traffic.  If we were going to set this up in production we would want to have a `REJECT`, or `DROP`  rule that looks something like this at the BOTTOM of our rules.

```
-A INPUT -j DROP
```

Remember IPTables is a first match engine so the order rules are in the file is the order they will be applied. If you have a `DROP` rule above the SSH or Puppet rule then it will never get to those and they won't work. 

# Lab Complete 