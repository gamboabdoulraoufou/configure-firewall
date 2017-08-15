### configure firewall with `firewalld`

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

The status will be running. This indicates that our firewall is up and running with the default configuration.
At this point, we will not enable the service. Enabling the service would cause the firewall to start up at boot. We should wait until we have created our firewall rules and had an opportunity to test them before configuring this behavior. This can help us avoid being locked out of the machine if something goes wrong.

