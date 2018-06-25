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


### Step 7: Create a new user named `grader` and give sudo permissions to it
At this point, user still logs in as `ubuntu` user.
1. Run `sudo adduser grader`.
2. Enter a password (twice) and fill out information for this new user.
   Now the `grader` user is created successfully.
In this project, the created user `grader` has a password `grader`.
3. Create a new file under the sudoers directory: `sudo vi /etc/sudoers.d/grader`.
   Add one line `grader ALL=(ALL:ALL) ALL` in this file and save by hitting the Esc key and then enter in “:” along with “wq”.
4. Verify the sudo permissions of user `grader`:
   * run `su - grader`
   * enter password
   * run `sudo -l`
   * enter password again
   If a line like the following should appear in the terminal, it means that the user `grader` has sudo permission:
   ```
   Matching Defaults entries for grader on ip-172-26-12-173.ec2.internal:
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

   User grader may run the following commands on ip-172-26-12-173.ec2.internal:
   (ALL : ALL) ALL
   ```
  
  
### Step 8: Log in to the AWS Lightsail instance with user `grader`
- On local machine: 
1. browse to directory `~/.ssh`. Run `ssh-keygen`.
2. Name the key pair, e.g. `grader_key`.
3. Enter in a passphrase twice. Two files will be generated, e.g. `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`.
4. Run `cat ~/.ssh/grader_key.pub` and copy the content in the file.
- On the grader's virtual machine:
1. From terminal on local machine, log in to the virtual machine with user `ubuntu` by running 
   `ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@54.173.124.133`.
2. In virtual machine, go to `grader`'s home directory by running: `cd /home/grader`.
3. Create a new directory named `.ssh` by running `mkdir .ssh`.
4. Create a new file `authorized_keys` by running `sudo vi authorized_keys` and then paste the copied content from `grader_key.pub` on local machine to this file `authorized_keys`. Save file `authorized_keys`.
5. On virtual machine, give permissions by running `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`.
6. Check in `/etc/ssh/sshd_config` file whether `PasswordAuthentication` is set to `no`. If `PasswordAuthentication` is not set to `no`, set it to `no` and save file.
7. Restart SSH with `sudo service ssh restart`.
- From terminal on local machine, log in to virtual machine with `grader` user by:
  `ssh -i ~/.ssh/grader_key -p 2200 grader@54.173.124.133`.

   


