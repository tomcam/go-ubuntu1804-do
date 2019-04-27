# go-ubuntu1804-do

Installing Golang on Ubuntu 18.04.02

# TODO:
* See if 1GB was enough to compile Go etc.
* Figure out whether to go through this again w/the default ssh key, since I already had one.

## Write down



| Note this             |  Write the value here      |
| --------------------- | -------------------------- |
| Droplet name          |                            |
| Droplet IP address    |                            |
| root username         |                            |
| root password         |                            |
| new username password |                            |
| new username password |                            |

## Creating the droplet

* From the green Create button, choose Droplets.
* From Ubuntu, choose 18.04.6 x64
* From Choose a plan, choose Starter, then $5/mo (or whatever)
* From Add backups, choose Yes (though not happening in this example)
* From Choose a datacenter region, choose the country closest to you.
* From Add your SSH keys, choose New SSH Key (or choose your existing one, as I did, with id_rsa.pub).
* From Finalize and create, choose 1 Droplet and consider changing the host name under Choose a hostname. I just called it dev.
* Choose Create.
* The Resources page displays 

## Setting up the root and new user
*  Drop into the terminal on your local machine
* Enter the following, replace the numbers with your own IP address.

```bash
ssh root@178.99.20.213
```
A message something like this appears:

```bash
The authenticity of host '178.99.20.213 (178.99.20.213)' can't be established.
ECDSA key fingerprint is SHA256:v5wGzOxtO/nI9rsXAfdkadfjtqntkQBTbrY9mjHQ.
Are you sure you want to continue connecting (yes/no)?
```

* Type `yes` and press Enter.

This happens:

```
Warning: Permanently added '178.99.20.213' (ECDSA) to the list of known hosts.
Connection closed by 178.99.20.213 port 22
```

* Run ssh again:

```bash
ssh root@178.99.20.213
```

This time you get in:

```bash
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-47-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Apr 27 08:58:53 UTC 2019

  System load:  0.0                Processes:           109
  Usage of /:   0.6% of 154.90GB   Users logged in:     0
  Memory usage: 1%                 IP address for eth0: 138.68.20.213
  Swap usage:   0%

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@dev:~#

```

## Create the new user

Let's call the new user `tom`. Write down the username and password in the chart above.


* Enter the following. Replace `tom` with whatever username you want to create.

```bash
adduser tom
```

* Copy the password you created to the system clipboard so you don't screw up entering it twice as you're about to do.

* Enter the password twice (just paste in) as requested.

```
Adding user `tom' ...
Adding new group `tom' (1000) ...
Adding new user `tom' (1000) with group `tom' ...
Creating home directory `/home/tom' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for tom
```

* Then just press Enter for everything else if you want:

```bash
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] y
```

## Update sshd_config

Edit the file `/etc/ssh/sshd_config` as shown:

```bash
sudo nano /etc/ssh/sshd_config
```

* Change  `PermitRootLogin prohibit-password` to `PermitRootLogin yes`
* Change `PasswordAuthentication no` to `PasswordAuthentication yes`

* Restart the ssh service:

```bash
sudo service ssh restart
```
For more information, see [Error Permission denied (publickey) when I try to ssh](https://www.digitalocean.com/community/questions/error-permission-denied-publickey-when-i-try-to-ssh)

## Reference

* [Initial Server Setup with Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04)
* [How To Set Up a Firewall with UFW on Ubuntu 18.04 ](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)
* [How To Install the Apache Web Server on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-18-04)

