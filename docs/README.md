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

## Create a password for root

Write down a password for root in the chart above and copy it to the system clipboard.

* Enter `passwd root` at the prompt:

```
passwd root
```

You're asked for the new password. 

* Paste it in twice:

```
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
```

## Update sshd_config

Edit the file `/etc/ssh/sshd_config` as shown:

```bash
sudo nano /etc/ssh/sshd_config
```

* Add or change `PermitRootLogin prohibit-password` to `PermitRootLogin yes` 
* Add or change `PasswordAuthentication no` to `PasswordAuthentication yes`

* Restart the ssh service:

```bash
sudo service ssh restart
```
For more information, see [Error Permission denied (publickey) when I try to ssh](https://www.digitalocean.com/community/questions/error-permission-denied-publickey-when-i-try-to-ssh)

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

## Set privileges for the new user

The new user needs to perform some adminstrative privileges. They're temporarily acquired through the
`sudo` command, but the user doesn't have the ability to use `sudo`.

* Add the user to the sudo group:

```
usermod -aG sudo tom
```

## Configure the UFW, the firewall

### The first rule: ensure you can ssh in

The first thing to do is to ensure you can use ssh to log back in!

```
ufw allow OpenSSH
```

It responds with:

```
Rules updated
Rules updated (v6)
```

### Turn on the firewall

* Enter `ufw enable` to activate the firewall:

```
ufw enable
```

You're asked:

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? 
```

* Type `y` and press Enter.

You're informed its working and will start up at boot from now on:

```
Firewall is active and enabled on system startup
```

### Review UFW's app profiles

* To see what profiles are set up, enter `ufw app list`:

```
ufw app list
```

You'll see something like this:

```
Available applications:
  OpenSSH
uwf app list
```

### Confirm the firewall's working:

* To see what the firewall is doing, enter `ufw status`:

```
ufw status
```

It informs you:

```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)   
```

### Copy authorized keys to the new user account.

* Run `rsync` as follows, **remembering to replace tom with the new user name*:

```
rsync --archive --chown=tom:tom ~/.ssh /home/tom
```

* Test it. Open a new terminal and try to ssh in to the new user account, remembering to replace `tom` with the new user name:

```
# Replace tom with the new username you created.
ssh tom@178.99.20.213
```

You get an introductory screen, similar to the one you got as root. Here it is, truncated:

```
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-47-generic x86_64)

 * Documentation:  https://help.ubuntu.com
...
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for detail
```

#### Notes

* See https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04 for more on this
* It may do the same as the `sshd_config` editing. Not sure. So try skipping this on next test.

## Install Go

* Return to the new user terminal (or ssh in again). In this example the user's name is tom:

```
# Not necessary if you're already in the terminal.
ssh tom@178.99.20.213
```
* Go to the home directory:

```
cd ~
```

* Visit the Go downloads page at [https://golang.org/dl/](https://golang.org/dl/) and figure out the most recent stable version. Let's say it's 1.12.4, so you'd do this:

```
curl -O https://dl.google.com/go/go1.12.1.linux-amd64.tar.gz
```

The progress looks something like this:

```
 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  121M  100  121M    0     0  92.2M      0  0:00:01  0:00:01 --:--:-- 92.2M
```

* Extract and copy to `/usr/local`:

```
sudo tar -xvf go1.12.1.linux-amd64.tar.gz -C /usr/local
```

There's a huge amount of output, greatly truncated here:

```
go/
go/AUTHORS
go/CONTRIBUTING.md
go/CONTRIBUTORS
go/LICENSE
go/PATENTS
go/README.md
go/VERSION
go/api/
go/api/README
go/api/except.txt
go/api/go1.1.txt
go/api/go1.10.txt
go/api/go1.11.txt
go/api/go1.12.txt
go/api/go1.2.txt
go/api/go1.3.txt
...etc
```

* Change permissions of the directory's owner and group to root:

```
sudo chown -R root:root /usr/local/go
```

Enter your password if asked.

* Create the recommended Go workspace (this may have been done)

```
mkdir -p $HOME/go/{bin,src}
```

* Open `~/.profile` for editing:

```
nano ~/.profile
```

* Add the following lines to `~/.profile` and save it:

```
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin:/usr/local/go/bin
```

* Quit the editor, then enter `. ~/.profile` at the command line to reload the environment:

```
. ~/.profile
```


Based on [How To Install Go and Set Up a Local Programming Environment on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-go-and-set-up-a-local-programming-environment-on-ubuntu-18-04)

## Reference

* [Initial Server Setup with Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04)
* [How To Set Up a Firewall with UFW on Ubuntu 18.04 ](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)
* [How To Install the Apache Web Server on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-18-04)

