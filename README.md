### Configure firewall with `firewalld`

Firewalld is a complete firewall solution available by default on CentOS 7 servers. 
In this post, I will show how to set up a firewall for your server and show you the basics of managing the firewall with the firewall-cmd administrative tool

> Turning on the Firewall

```sh
# log as root
sudo su - root

# turn the daemon on
sudo systemctl start firewalld.service

# check firewall status
firewall-cmd --state

```

The status will be `running`. This indicates that our firewall is up and running with the default configuration.
At this point, we will not enable the service. Enabling the service would cause the firewall to start up at boot. We should wait until we have created our firewall rules and had an opportunity to test them before configuring this behavior. This can help us avoid being locked out of the machine if something goes wrong.

> Exploring zones

The firewalld daemon manages groups of rules using entities called "zones". Zones are basically sets of rules dictating what traffic should be allowed depending on the level of trust you have in the networks your computer is connected to. Network interfaces are assigned a zone to dictate the behavior that the firewall should allow.

The pre-defined zones within firewalld are:

- drop: The lowest level of trust. All incoming connections are dropped without reply and only outgoing connections are possible.
- block: Similar to the above, but instead of simply dropping connections, incoming requests are rejected with an icmp-host-prohibited or icmp6-adm-prohibited message.
- public: Represents public, untrusted networks. You don't trust other computers but may allow selected incoming connections on a case-by-case basis.
- external: External networks in the event that you are using the firewall as your gateway. It is configured for NAT masquerading so that your internal network remains private but reachable.
- internal: The other side of the external zone, used for the internal portion of a gateway. The computers are fairly trustworthy and some additional services are available.
- dmz: Used for computers located in a DMZ (isolated computers that will not have access to the rest of your network). Only certain incoming connections are allowed.
work: Used for work machines. Trust most of the computers in the network. A few more services might be allowed.
- home: A home environment. It generally implies that you trust most of the other computers and that a few more services will be accepted.
- trusted: Trust all of the machines in the network. The most open of the available options and should be used sparingly.
To use the firewall, we can create rules and alter the properties of our zones and then assign our network interfaces to whichever zones are most appropriate

```sh 
 # get default zones
 firewall-cmd --get-default-zone
 
 # get interfaces managed on active zone
 firewall-cmd --get-active-zones
 
 # get rules associated on each active zone
 firewall-cmd --list-all
 
 # get list of available zones
 firewall-cmd --get-zones
 
 # get rules associated on specific zone (internal in this case)
 firewall-cmd --zone=internal --list-all
 
 # get rules associated on each zone
 firewall-cmd --list-all-zones
 
 ```
 
 > Selecting Zones for your Interfaces

```sh
# transit eth1 to internal zone
firewall-cmd --zone=internal --change-interface=eth1

# check rules associated on each zone to check which zone manage eth1
firewall-cmd --list-all-zones
 
# check active zone, you should see public and internal
firewall-cmd --get-active-zones
 
# restart firewall services, all interface will be int default zone
systemctl restart firewalld.service
 
# check active zone, you should see interfaces eth0 and eth1 are in the same zone: public
firewall-cmd --get-active-zones
 
```


> Changing the Zone of your Interface Permanently  

Interfaces will always revert to the default zone if they do not have an alternative zone defined within their configuration. On CentOS, these configurations are defined within the `/etc/sysconfig/network-scripts` directory with files of the format `ifcfg-interface`.

```sh
# edit config file
vi /etc/sysconfig/network-scripts/ifcfg-eth1

# add configuration
...
DNS1=2001:4860:4860::8844
DNS2=2001:4860:4860::8888
DNS3=8.8.8.8
ZONE=internal

# restart network service
sudo systemctl restart network.service

# restart firewall service
sudo systemctl restart firewalld.service

# check active zone
firewall-cmd --get-active-zones

```

You can get more details about each of these services by looking at their associated .xml file within the /usr/lib/firewalld/services directory


> Adjusting the Default Zone

If all of your interfaces can best be handled by a single zone, it's probably easier to just select the best default zone and then use that for your configuration.

```sh
# change default zone
sudo firewall-cmd --set-default-zone=home

```

> Setup web server to test our firewall

```sh
# install http server
yum -y install httpd

# install wget for test
yum install -y wget 

# start http service
systemctl enable httpd
systemctl start httpd
systemctl status httpd

# create web page for test
mkdir -p /var/www/html/
echo "Hello world" > /var/www/html/index.html

# go to http:[your_public_ip]
# you can see you don't have access yet. so you need to configure firewall to allow http or open port 80.
```


> Setting Rules for your Applications: adding a Service to your Zones

```sh
# list services manged by firewall
firewall-cmd --get-services

# chekc public zone services
firewall-cmd --zone=public --list-services

# add http service into public zone
sudo firewall-cmd --zone=public --add-service=http --permanent # add permanent rule
sudo firewall-cmd --zone=public --add-service=http # add temporary rule. Role will e lost after reboot

# chekc again public zone services. You should http
firewall-cmd --zone=public --list-services

```

> What If No Appropriate Service Is Available?  

The firewall services that are included with the firewalld installation represent many of the most common requirements for applications that you may wish to allow access to. However, there will likely be scenarios where these services do not fit your requirements.

In this situation, you have two options.

Opening a Port for your Zones
The easiest way to add support for your specific application is to open up the ports that it uses in the appropriate zone(s). This is as easy as specifying the port or port range, and the associated protocol for the ports you need to open.

```sh
# open port
firewall-cmd --zone=internal --add-port=80/tcp

# list port 
firewall-cmd --list-ports

```


> Remove service or port

```sh
# remove service
firewall-cmd --zone=public --remove-service=http

# remove port
firewall-cmd --zone=internal --remove-port=80/tcp
```


> Enable Your Firewall to Start at Boot

```sh
sudo systemctl enable firewalld
```

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7 
 
 
 
 
 
 
 ```
