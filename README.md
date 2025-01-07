# Security Basics
## Security basics for a fresh Linux install

Author [https://github.com/Vince-0](https://github.com/Vince-0/Projects)

My notes for basic security configurations on any fresh Linux server. I like to use Debian.


## User

Add a new user. You shouldnt be using root to log in directly

`adduser mynewuser`

Add your user to the sudoers file for the appropriate permissions

`nano /etc/sudoers`

Add the line under the "User privilege specification" comment 

`mynewuser ALL=(ALL:ALL) ALL`


## SSH

Edit sshd config

`nano /etc/ssh/sshd_config`

Or rather create a custom file

`nano /etc/ssh/sshd_config.d/10_custom.conf`

Add the line to disallow root login via SSH

`PermitRootLogin prohibit-password`

On your client computer generate a public SSH key file for your current user

`ssh-keygen`

Copy the SSH key file to your server

`ssh-copy-id mynewuser@[myserver]`

Test that you can log in without passwords before restarting sshd service

`systemctl restart sshd`

Check that sshd restarted without any errors and its Active status is "active (running)"

`systemctl status sshd`

For a more secure SSH server disallow password authentication and use only key files
```
PasswordAuthentication no
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
PermitRootLogin no
```

## Firewall
Debian has iptables installed by default but it allows all connections.
Rules are evaluated from top to bottom so allow rules should be followed by block rules.

Allow remote IP or subnet with CIDR notation

`iptables -A INPUT -s [IP-GOES-HERE] -j ACCEPT`

Allow established sessions

`iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT`

Block port numbers for common services like SSH 22 tcp, FTP 21 tcp, MySQL 3306 tcp, SIP 5060 udp

`iptables -A INPUT -p [tcp|udp] --destination-port [PORT-NUMBER-HERE] -j DROP`

Set defaults to drop all other connections

```
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

View current rules

`iptables -nL`

Save

`iptables-save > /etc/iptables.save`

Restore

`iptables-restore /etc/iptables.save`

Auto restore on boot by installing iptables-persistent. It will ask you to save your current rules.

`apt install iptables-persistent`


## Fail2Ban
[Fail2Ban](https://github.com/fail2ban/fail2ban) 

"Fail2Ban scans log files like /var/log/auth.log and bans IP addresses conducting too many failed login attempts."

Install 

`apt install fail2ban`

Enable systemd log for fail2ban to check for sshd failed attempts

`echo "sshd_backend = systemd" >> /etc/fail2ban/paths-debian.conf`

Jail default settings can be set in /etc/fail2ban/jail.conf

Set ban time to 5 days

`bantime = 7200m`

Restart fail2ban

`systemctl restart fail2ban`

Check service status
`systemctl status fail2ban`

Check the logs
`tail -f -n 200 /var/log/fail2ban.log`

Fail2ban will add a chain f2b-sshd and block any offending source IPs with a target of REJECT
`iptables -nL`

