# Linux Server Configuration
A baseline installation of a Linux distribution on a virtual machine and prepare it to host the web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Project Tools Description 
- Public IP address: 54.173.124.133
- Acessible SSH port: 2200
- Application URL: http://54.173.124.133.xip.io/ 
- Linux version: Ubuntu 16.04.4 LTS
- virtual private server: [Amazon Lightsail](https://lightsail.aws.amazon.com)
- Deployed web application: [Item Catalog](https://github.com/blueblackw/item-catalog)
- Database server: [PostgreSQL](https://www.postgresql.org)
- Local machine: MacBook Pro

## Configuration Steps
### Step 1: Create an instance with Amazon Lightsail
1. Log into [Amazon Lightsail]((https://lightsail.aws.amazon.com)) with an Amazon Web Service account. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
2. Follow the 'Create an instance' link.
3. Under "Create an instance", select platform as " Linux/Unix" and select a blueprint as "OS Only". Then under the options "OS Only", click to select "Ubuntu" as the operating system. 
4. Choose the payment plan. Name the instance in the textbox on the bottom of the page. Click "Create".
5. Wait for the instance to start up.
6. After the instance is running, click on the instance to enter the instance home. Under "Network" panel of the instance, click the button "Create static IP". This is to prevent the ip to change every time you restart the instance.

**Note:** For setting up OAuth for the application in this project, we need a DNS name that refers to the instance's IP address. We use the [xip.io](http://xip.io/) service to get one; this is a public service offered for free by Basecamp. For instance, the DNS name 54.173.124.133.xip.io refers to the server above.

#### Reference 
- [ServerPilot](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html)


### Step 2: SSH into the server
1. From the Account menu on Amazon Lightsail, click on SSH keys tab and download the Default Private Key.
2. Move this private key file named like LightsailDefaultPrivateKey-.pem to the local folder ~/.ssh and rename it lightsail_key.rsa.
3. In the terminal of local machine, type `chmod 600 ~/.ssh/lightsail_key.rsa`.
4. Connect tothe instance from termial by: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.59.39.163`


### Step 3: Update and upgrade installed packages
1. Notify the system of what package updates are available: run `sudo apt-get update` in terminal.
2. Download available package updates: run `sudo apt-get upgrade` in terminal.

### Step 4: Change the SSH port from 22 to 2200
1. Open file /etc/ssh/sshd_config with command `vi /etc/ssh/sshd_config`.
2. Change the port number on line 5 to `2200`. Save the change.
3. Restart SSH: `sudo service ssh restart`.

### Step 5: Configure the Uncomplicated Firewall (UFW)
1. Check to see the firewall status: `sudo ufw status`. It should look like this:
2. Set the ufw firewall to block everything coming in: `sudo ufw default deny incoming`.
3. Set the ufw firewall to allow everything outgoing: `sudo ufw default allow outgoing`.
4. Set the ufw firewall to allow SSH: `sudo ufw allow ssh`.
5. Allow all tcp connections for port `2200` so that SSH will work: `sudo ufw allow 2200/tcp`.
6. Set the ufw firewall to allow a basic HTTP server: `sudo ufw allow www`.
7. Set the ufw firewall to allow NTP: `sudo ufw allow 123/udp`.
8. Set the ufw firewall to deny port `22`: sudo ufw deny 22.
9. Enable the ufw firewall: `sudo ufw enable`.
10. Check ufw status again: `sudo ufw status`. If done correctly, it should look like this:
```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)
```
11. Update the external (Amazon Lightsail) firewall on the browser: 
- Click on the 'Manage' option of the Amazon Lightsail Instance.
- Select "Networking" tab.
- Change the firewall configuration to match the internal firewall settings above: only ports `80`(TCP), `123`(UDP), and `2200`(TCP) should be allowed. Deny the default port `22`.
12. From terminal on local machine, log in to the ubuntu instance by running:
`ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@54.173.124.133`

**Note:** Connecting to the instance through a browser no longer works because Lightsail's browser-based SSH access only works through port 22, which is now denied.

#### Reference 
- [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04)

### Step 6: Use Fail2Ban to ban attackers
Install fail2ban in order to mitigate brute force attacks by users and bots alike.
1. `sudo apt-get update`.
2. `sudo apt-get install fail2ban`
3. Install the sendmail package to send the alerts to the admin user: `sudo apt-get install sendmail`
4. Create a copy of a file: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
5. Open the jail.local and edit it: `sudo vi /etc/fail2ban/jail.local`. Change the *destemail* field `useremail@domain` in `destemail = useremail@domain` to admin user's email address.
#### Reference
- [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)

### Step 7: Create a new user named `grader` and give sudo permissions
1. 
