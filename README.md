# Documentation-Open-SSH-Setup 
https://wiki.debian.org/SSH
* **Use** SSHv2 -> Set "Protocol 2" in the sshd config file
* Do **NOT** allow password login -> *use SSH Keys.*
* Do **NOT** allow root login. -> *create a user with sudo privileges.*
* All changes made to the config file need "systemctl reload sshd" to force the configuration to be reloaded.

## Get help
* man ssh
* man sshd
* man sshd_config
  
## Preparation
```
sudo apt update && sudo apt upgrade --show-upgraded
```
## Installation

### Server:
```
sudo apt install openssh-server
```
### Client
```
sudo apt install openssh-client
```

## Configuration

### Create a new user on your SSH server
https://manpages.debian.org/bullseye/passwd/useradd.8.en.html
https://wiki.debian.org/sudo
```
sudo useradd sshuser
sudo passwd donotforget2changethis
sudo echo 'sshuser ALL=(ALL) ALL' >> /etc/sudoers
```
Check that you can switch to the root user with ```sudo su```
```
vi /etc/ssh/sshd_config
```
#### Controlling Root Logins
Change "#PermitRootLogin yes" to "PermitRootLogin no" then restart the ssh-server.
You could allow root to login with key with "PermitRootLogin without-password" to only allow login with a key instead if you need.

#### Controlling SSH access
* Allow access:
  * by user: Add "AllowUsers user1 uer2 userN" in the config file.
  * by group: Add "AllowGroups group1 group2 groupN" in the config file.
* Deny access:
  * by user: Add "DenyUsers user1 uer2 userN" in the config file.
  * by group: Add "DenyGroups group1 group2 groupN" in the config file.

#### Disable TCP Port Forwarding
Add "AllowTcpForwading no" to the config file to prevent security issues.
Or at least add "GatewayPorts no" if you need Port forwading. This setting blocks remote requests to the forwarded ports.
Use a local firewall on the host to deny access to unused ports.

##### SSH Port Forwarding
You can port forwarding if you need access to a port that is only listening to the local loopback address of the host.
That way you can use a client on your local machine to send encrypterd requests with the apropiate port through the ssh conection to the host server.
The downside is that there is a reason the host has configured the port only to listen oon the local loopback adress. 
So it is important to configure the clients firewall so that nobody can connect TO the client on that forwarded port to get access to the server trough the forwarded port.

##### Dynamic Port Forwarding /SOCKS
Tunnel Traffic through a ssh connection to bind the visible origin of requests to the server instead of the client.
* Single Client: ssh -L 8080:google.com:80 host
* Multiple Clients: ssh -D 8080 host (You have to configure your webbrowser to use a SOCKS Proxy. use 127.0.0.1:8080 an your client)

##### Reverse Port Forwarding
Use a local firewall on your host server unused ports to prevent missuse
ssh -R 2222:127.0.0.1:22 

#### Bind SSH to specific adress
If your ssh server has access to different networks add "ListenAdress host_or_adress1" to the config file. You can add multiple lines if you need to listen on different networks.
Like that you can only listen on private networks and ignore public networks.

#### Change the Default Port (e.g. 2222)
This can reduce the number of unwanted connections but with a port scan it is an easy task for an attacker to define the new port. Add "Port 2222" to the config file.

#### Disable the Banner
To reduce information leakage add "Banner none" to the config file. If you need to use a banner try not to add more information than needed (Banner /etc/issue.net).

```
ssh -L 5432:127.0.0.1:5432 host
```
### Generate a SSH key on your client
https://manpages.debian.org/bullseye/openssh-client/ssh-keygen.1.en.html

Use a phassphrase for the private key. That adds an additional security layer.
```
ssh-keygen
```
Add the public key to the remote host.
```
ssh-copy-id -i ~/.ssh/id_rsa.pub hostUser@host
```
This will add the key to *~/.ssh.authorized_keys* on the host.
To disable login again remove key from that file.

### Login to the server from your client
```
ssh $remote_user@$remote_host
```

### Disable SSH password login on the server
If the ssh login with the key worked, turn off password authentication completly. 

```
vi /etc/ssh/sshd_config
```
Change "#PasswordAuthentication yes" to "PasswordAuthentication no" then restart the ssh-server.
```
sudo systemctl restart sshd
```
### Check service status
https://manpages.debian.org/bullseye/systemd/systemctl.1.en.html
```
systemctl status sshd
```

```
sudo systemctl reboot
```

### Lock Out Attackers with fail2ban
https://packages.debian.org/bullseye/fail2ban
#### Install
```
sudo apt install fail2ban
```
#### Check service status
```
systemctl status fail2ban
```
#### Customize configuration
```
cd /etc/fail2ban
head -20 jail.conf
```
```
sudo cp jail.conf jail.local
vi jail.local
```

### Optional white-/black listing (only recommended with static client ip address)
```
echo 'sshd: your.static.ip.address L' >> /etc/hosts.allow
```
```
echo 'sshd: ALL' >> /etc/hosts.deny
```
```
sudo systemctl restart sshd
```
### Optional Set Up Two Factor Authentication (with Google Authenticator)
