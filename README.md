# Documentation-Open-SSH-Setup 
https://wiki.debian.org/SSH

* Do **NOT** allow password login -> *use SSH Keys.*
* Do **NOT** allow root login. -> *create a user with sudo privileges.*
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
Change "#PermitRootLogin yes" to "PermitRootLogin no" then restart the ssh-server.

### Generate a SSH key on your client
https://manpages.debian.org/bullseye/openssh-client/ssh-keygen.1.en.html
```
ssh-keygen
```
```
ssh-copy-id -i ~/.ssh/id_rsa.pub hostUser@host
```

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
