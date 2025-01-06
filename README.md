# Security-Basics
## Security basics for any fresh installs

Author [https://github.com/Vince-0](https://github.com/Vince-0/Projects)

My notes for basic security configurations on any fresh Linux server.

#### User:
Add a new user. You shouldnt be using root to log in directly

`adduser mynewuser`

Add your user to the sudoers file for the appropriate permissions

`nano /etc/sudoers`

Add the line under the "User privilege specification" comment 

`mynewuser ALL=(ALL:ALL) ALL`

#### SSH:
Edit sshd config

`nano /etc/ssh/sshd_config`

Or rather create a custom file

`nano /etc/ssh/sshd_config.d/10_custom.conf`

Add the line to disallow root login via SSH

`PermitRootLogin prohibit-password`

For preferred key security remove password log in altogether and use only key files

`PasswordAuthentication no`

On your client computer generate a public SSH key file for your current user

`ssh-keygen`

Copy the SSH key file to your server

`ssh-copy-id mynewuser@[myserver]`

Test that you can log in without passwords before restarting sshd service

`systemctl restart sshd`

Check that sshd restarted without any errors and its Active status is "active (running)"

`systemctl status sshd`

#### Firewall:
