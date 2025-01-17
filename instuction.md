# born2beroot
## Table of Contents
> 1. [Installation](#installation)
> 2. [Configuration](#configuration)
> - [Installing sudo](#installing_sudo)
> - [Installing tools](#installing_tools)
> - [Installing SSH and configuring SSH service](#install_ssh)
> - [Installing and configuring UFW (Uncomplicated Firewall)](#ufw)
> - [Connecting SSH-server](#connect_ssh)
> - [Set password policy (source)](#password_policy)
> - [Greating a New Group](#new_group)
> - [Creating a New User and assign into group](#new_user)
> - [Configuring sudoers group](#sudoers)
> - [Deletion users and groups](#del)
> - [Crontab configuration](#crontab)
> - [Change hostname (!!!This is for when you defend!!!)](#host)
## 1. <a name="installation"></a>Installation
## 2. <a name="configuration"></a>Configuration
### <a name="installing_sudo"></a>2.1. Installing sudo
Logit as root
```
$ su -
```
Install sudo
```
$ apt-get update -y
$ apt-get upgarde -y
$ apt install sudo
```
Add user in sudo group
```
$ usermod -aG sudo your_username
```
Check if user in sudo group
```
getent group sudo
```
Give privilege as a su.
Open sudoers file:
```
$ sudo visudo
```
Add this line in file:
```
your_username ALL=(ALL) ALL
```
### <a name="installing_tools"></a>2.2. Installing tools
Installing git
```
$ apt-get update -y
$ apt-get upgrade -y
$ apt-get install git -y
```
Check git version
```
git --version
```
Installing wget (wget is a free and open source tool for downloading files from web repositories.)
```
$ sudo apt-get install wget
```
Installing Vim
```
$ sudo apt-get install vim
```
### <a name="install_ssh"></a>2.3. Installing SSH and configuring SSH service
Installing
```
$ sudo apt-get update
$ sudo apt install openssh-server
```
Check the SSH server status
```
$ sudo systemctl status ssh
```
Restart the SSH service
```
$ sudo service ssh restart
```
Changing default port (22) to 4242
```
$ sudo nano /etc/ssh/sshd_config
```
Edit the file change the line #Port22 to Port 4242. Find thid line:
```
#Port 22
```
*Change it like this:*
```
Port 4242
```
Check if port settings got right
```
$ sudo grep Port /etc/ssh/sshd_config
```
To disable SSH login as root irregardless of authentication mechanism, replace below line
```
32 #PermitRootLogin prohibit-password
```
with:
```
32 PermitRootLogin no
```
Restart the SSH service
```
$ sudo service ssh restart
```
### <a name="ufw"></a>2.4. Installing and configuring UFW (Uncomplicated Firewall)
Install UFW
```
$ sudo apt-get install ufw
```
Enable
```
$ sudo ufw enable
```
Check the status
```
$ sudo ufw status numbered
```
Configure the rules
```
$ sudo ufw allow ssh
```
Configure the port rules
```
$ sudo ufw allow 4242
```
Delete the new rule: (This is for when you defend your Born2beroot)
```
$ sudo ufw status numbered
$ sudo ufw delete (that number, for example 5 or 6)
```

### <a name="connect_ssh"></a>2.5. Connecting SSH-server
Add forward rule for VirtualBox.
Restart SSH server (go to the your VM machine)
```
$ sudo systemctl restart ssh
```
Check ssh status:
```
$ sudo service sshd status
```
From host side from iTerm2 or Terminal enter as shown below:
```
$ ssh your_username@127.0.0.1 -p 4242
```
Quit the connection:
```
$ exit
```
### <a name="password_policy"></a>2.6. Set password policy (source)
#### Password Age
Configure password age policy via sudo nano ```/etc/login.defs```.
```
$ sudo nano /etc/login.defs
```
To set password to expire every 30 days, replace below line
```
160 PASS_MAX_DAYS   99999
```
with
```
160 PASS_MAX_DAYS   30
```
To set minimum number of days between password changes to 2 days, replace below line
```
161 PASS_MIN_DAYS   0
```
with
```
161 PASS_MIN_DAYS   2
```
To send user a warning message 7 days (defaults to 7 anyway) before password expiry, keep below line as is.
```
162 PASS_WARN_AGE   7
```
#### Password Strangth
Installing password quality checking library (libpam-pwquality):
```
$ sudo apt-get install libpam-pwquality
```
Verify whether libpam-pwquality was successfully installed
```
$ dpkg -l | grep libpam-pwquality
```
Configure password strength policy nano ```sudo vi /etc/pam.d/common-password```, specifically the below line:
```
$ sudo vi /etc/pam.d/common-password
<~~~>
25 password        requisite                       pam_pwquality.so retry=3
<~~~>
```
To set password minimum length to 10 characters, add below option to the above line.
```
minlen=10
```
To require password to contain at least an uppercase character and a numeric character:
```
ucredit=-1 dcredit=-1
```
To set a maximum of 3 consecutive identical characters:
```
maxrepeat=3
```
To reject the password if it contains ```<username>``` in some form:
```
reject_username
```
To set the number of changes required in the new password from the old password to 7:
```
difok=7
```
To implement the same policy on root:
```
enforce_for_root
```
Finally, it should look like the below:
```
password    requisite   pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```
### <a name="new_group"></a>2.7. Greating a New Group
Create new *user42* group
```
$ sudo addgroup user42
```
Check if group created:
```
$ getent group
```
### <a name="new_user"></a>2.8. Creating a New User and assign into group
Check the all local users:
```
$ cut -d: -f1 /etc/passwd
```
Create the user
```
$ sudo adduser new_username
```
Assign an user into “evaluating” group (This is for when you defend)
```
$ sudo usermod -aG user42 your_username
$ sudo usermod -aG evaluating your_new_username
```
Check if the user is in group
```
$ getent group user42
$ getent group evaluating
```
Check which groups user account belongs:
```
$ groups
```
Check if password rules working in users:
```
$ sudo chage -l your_new_username
```
### <a name="sudoers"></a>2.9. Configuring sudoers group
Make dir ```/var/lod/sudo```
Go to file:
```
$ sudo visudo /etc/sudoers
```
Add following for authentication using sudo has to be limited to 3 attempts in the event of an incorrect password:
```
Defaults     secure_path="..."
Defaults     passwd_tries=3
```
For wrong password warning message, add:
```
Defaults     badpass_message="Password is wrong, please try again!"
```
Each action log file has to be saved in the /var/log/sudo/ file:
```
Defaults    logfile="/var/log/sudo/sudo.log"
```
To archive all sudo inputs & outputs to ```/var/log/sudo/```:
```
Defaults    log_input,log_output
Defaults    iolog_dir="/var/log/sudo"
```
To require TTY:
```
Defaults        requiretty
```
To set sudo paths to
```
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```
![image](https://user-images.githubusercontent.com/84144634/141363542-b1f5ca15-e981-4379-9ffb-ea062eb8856f.png)

### <a name="del"></a>2.10. Deletion users and groups
Delete user (-r - delete home directory and user's files):
```
$ sudo userdel -r user_name
```
Delete group:
```
$ sudo groupdel group_name
```
### <a name="crontab"></a>2.11. Crontab configuration
Install the netstat tools (need for script monitoring.sh)
```
$ sudo apt-get update -y
$ sudo apt-get install -y net-tools
```
Open crontab and add the rule (- u user (specific user) ; - e (editing or creating the current schedule)):
```
sudo crontab -u root -e
```
To schedule a shell script to run every 10 minutes, replace below line
```
23 */10 * * * * sh /path/to/script
```
Check crontab
```
sudo crontab -u root -l
```
### <a name="host"></a>2.12. Change hostname (!!!This is for when you defend!!!)
Check current hostname
```
$ hostnamectl
```
Change the hostname
```
$ hostnamectl set-hostname new_hostname
```
Change /etc/hosts file
```
$ sudo nano /etc/hosts
```
Change old_hostname with new_hostname:
```
127.0.0.1       localhost
127.0.0.1       new_hostname
```
Reboot and check the change
```
$ sudo reboot
```
