# server-installation
cheat sheet 

used sources:
- https://beyond.lol/ubuntu-20-04-lts-automatische-updates-aktivieren/
- https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-as-a-send-only-smtp-server-on-ubuntu-20-04
- https://tonyteaches.tech/postfix-gmail-smtp-on-ubuntu/

## Install all nessecary packages:
- apt-listchanges : receive list changes
- unattended-upgrades : automatic updates
- mailutils : sending mails from server

~~~~
sudo apt install apt-listchanges unattended-upgrades mailutils
~~~~
## 1. set up unattended-upgrades:

should start automatically, if not:
~~~~
sudo dpkg-reconfigure -plow unattended-upgrades
~~~~
configure unattended-upgrades:
~~~~
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
~~~~

adjust/uncomment following lines:
~~~~
        "${distro_id}:${distro_codename}-updates";
        
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Mail „info@example.com“;
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot „true“;
Unattended-Upgrade::Automatic-Reboot-WithUsers "true";
Unattended-Upgrade::Automatic-Reboot-Time „02:00“;
~~~~
Receive list changes via mail. 
Add your email:
~~~~
nano /etc/apt/listchanges.conf
~~~~
test the setup and verify logs:
~~~~
sudo unattended-upgrades --dry-run
sudo cat /var/log/unattended-upgrades/unattended-upgrades.log
~~~~
## 2. setup postfix for GMAIL / Googlemail
The default option is Internet Site. That’s the recommended option for your use case, so press TAB, and then ENTER. If you only see the description text, press TAB to select OK, then ENTER.

If it does not show up automatically, run the following command to start it:
~~~~
sudo dpkg-reconfigure postfix
~~~~
Get login credentials at Googlemail.
**Create / use app-password for this service!**

Create a file with credentials:
~~~~
sudo nano /etc/postfix/sasl/sasl_passwd
~~~~
Add your credentials to the file:
~~~~
[smtp.gmail.com]:587 YOUR_EMAIL@gmail.com:YOUR_PASSWORD
~~~~
Next, create a hash database file with the following postmap command:
~~~~
sudo postmap /etc/postfix/sasl/sasl_passwd
~~~~
Note that now you will have a database file at /etc/postfix/sasl/sasl_passwd.db 

In order to protect the plain-text password, change the owner and permission for the SASL files as follows:
~~~~
sudo chown root:root /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db
sudo chmod 0600 /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db
~~~~

Edit configuration file:
~~~~
sudo nano /etc/postfix/main.cf
~~~~

comment following line:
~~~~
#smtpd_tls_security_level=may
#smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
~~~~
add the following lines:
~~~~
relayhost = [smtp.gmail.com]:587
smtp_tls_security_level=encrypt
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
~~~~
Restart Postfix to apply our configuration changes:
~~~~
sudo systemctl restart postfix
~~~~

FINISH
