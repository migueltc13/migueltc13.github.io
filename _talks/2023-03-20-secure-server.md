---
title: "Linux Server Hardening"
collection: talks
type: "Talk"
permalink: /talks/secure-linux-server
venue:
date: 2023-03-20
location: "Online"
---

Securing a server by implementing various measures to protect it from unauthorized access and potential cyber-attacks. In this guide, we will go through ten important steps to harden a Linux server.

### 1. Enable automatic updates

`apt install unattended-upgrades`
`dpkg-reconfigure --priority=low unattended-upgrades`


### 2. Lower privileges users

`adduser <USERNAME>`
`usermod -aG sudo <USERNAME>`

Tip: restrict yourself to the necessary privileges when connected to your server.


### 3. Use RSA private key to login

(On server)

`mkdir ~/.ssh && chmod 700 ~/.ssh`

(On local machine)

To generate the RSA key pair run:

`ssh-keygen -b 4096`

Copy the public key to your server with:

`ssh-copy-id <USERNAME>@<IP>`

or

`scp ~/.ssh/id_rsa.pub <USERNAME>@<IP>`


### 4. Lock down password logins

`sudo nano /etc/ssh/sshd_config`

- a) Change “Port”
- b) Change to “AddressFamily inet” (ipv4 only)
- c) “PermitRootLogin no”
- d) “PasswordAuthentication no”

`sudo systemctl restart sshd`


### 5. Enable firewall

`sudo ss -tupln`
`sudo apt install ufw`
`sudo ufw allow <SSH_PORT> ...`
`sudo ufw enable`
`sudo ufw status`


### 6. Block pings

`sudo nano /etc/ufw/before.rules`

Add this line to “ok icmp codes for INPUT”:

`-A ufw-before-input -p icmp --icmp-type echo-request -j DROP`

`sudo ufw reload` or `sudo reboot`


### 7. Disable unused services and ports

Check which services and ports are currently open on your server by running:

`sudo netstat -tulpn`

Then, disable any unused services and ports by editing the appropriate configuration files.

For example, if you're not using FTP, you can disable it by commenting out the following line in the `/etc/ssh/sshd_config` file:

`#Subsystem sftp /usr/lib/openssh/sftp-server`

Make sure to restart any services you've disabled.


### 8. Install and configure fail2ban

Fail2ban is a popular tool for preventing brute-force attacks on your server. It monitors your server's logs and blocks IP addresses that repeatedly fail authentication attempts. To install fail2ban, run:

`sudo apt install fail2ban`

Then, create a new configuration file for your service.

For example, to protect your SSH server, create a file named `/etc/fail2ban/jail.local` with the following contents:

```
[sshd]
enabled = true
port = <SSH_PORT>
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

This configuration will block any IP address that fails to authenticate more than 3 times within an hour.

Finally, restart fail2ban to apply the new configuration:

`sudo systemctl restart fail2ban`


### 9. Install and configure a web application firewall (WAF)

If you're running a web server, it's a good idea to install a WAF to protect your site from common web attacks.

One popular WAF for Linux is ModSecurity. To install ModSecurity and the Apache module, run:

`sudo apt install libapache2-modsecurity`

Then, enable the module and restart Apache:

`sudo a2enmod mod-security`
`sudo systemctl restart apache2`

You'll also need to create a configuration file for ModSecurity. The default configuration file is located at `/etc/modsecurity/modsecurity.conf`, but you may want to create a separate file for your site's specific rules.


### 10. Monitor server logs for suspicious activity

Finally, it's important to regularly monitor your server's logs for any signs of suspicious activity.

You can use tools like logwatch or logrotate to automatically archive and analyze your logs, or set up a more sophisticated logging system like ELK (Elasticsearch, Logstash, and Kibana) to track and visualize your server's activity in real time.
