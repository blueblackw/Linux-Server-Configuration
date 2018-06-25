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
**From now on, user logs in to virtual machine as user `grader`.**
#### Reference
[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
[Ubuntu documentation](https://help.ubuntu.com/community/SSH/OpenSSH/Keys)


### Step 9: Configure the local timezone to UTC.
1. Run `sudo dpkg-reconfigure tzdata`.
2. In the pop up console, select `None of the above`.
3. Select `UTC` and confirm. User will see 
   ```
   Current default time zone: 'Etc/UTC'
   Local time is now:      Mon Jun 25 02:17:24 UTC 2018.
   Universal Time is now:  Mon Jun 25 02:17:24 UTC 2018.
   ```
#### Reference
[Ubuntu documentation](https://help.ubuntu.com/community/UbuntuTime)
[Ask Ubuntu](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)


### Step 10: Install and configure Apache to serve a Python mod_wsgi application
- Install and configure Apache
1. Run `sudo apt-get install apache2`.
2. Enter the public IP of the Amazon Lightsail instance as a URL in a browser. If Apache is working fine, a page with the title 'Apache2 Ubuntu Default Page' should show.
- Install mod_wsgi
1. Install the Python 3 mod_wsgi package with command: 
   
   `sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable `mod_wsgi` by running: `sudo a2enmod wsgi`.


### Step 11: Install and configure PostgreSQL
1. Install PostgreSQL: `sudo apt-get install postgresql`.
2. Open file `/etc/postgresql/9.5/main/pg_hba.conf` with command `sudo vi /etc/postgresql/9.5/main/pg_hba.conf`.
3. There should be content in file `/etc/postgresql/9.5/main/pg_hba.conf` as:
   ```
   local   all             postgres                                peer
   local   all             all                                     peer
   host    all             all             127.0.0.1/32            md5
   host    all             all             ::1/128                 md5
   ```
4. Switch to Linux user `postgres` by running `sudo su - postgres`. Only use this user `postgres` for accessing PostgreSQL.
5. Connect to psql for interacting with PostgreSQL by running: `psql`.
6. Create a user `catalog` with a password `catalog` (or other passwords) by running:
   ```
   CREATE USER catalog WITH PASSWORD 'catalog';
   ```
7. Give user `catalog` the CREATEDB capability:
   ```
   ALTER USER catalog CREATEDB;
   ```
8. Check whether the user `catalog` is created with correct privileges by running `\du`. The returned results should look like:
   ```
                                     List of roles
    Role name |                         Attributes                         | Member of 
   -----------+------------------------------------------------------------+-----------
    catalog   | Create DB                                                  | {}
    postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
   ```
9. Create a database named catalog owned by catalog user:
   ```
   CREATE DATABASE catalog WITH OWNER catalog;
   ```
10. Connect to the database: `\c catalog`.
11. Revoke all rights: `REVOKE ALL ON SCHEMA public FROM public;`.
12. Lock down the permissions to only let catalog role create tables: `GRANT ALL ON SCHEMA public TO catalog;`.
13. Log out from PostgreSQL: `\q`. 
14.Then return to the `grader` user: `exit`.
#### Reference
[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

### Step 12: Install git and clone the project Item Catalog
User now logs in as `grader` user.
1. Install git: `sudo apt-get install git`.
2. Create a directory: `/var/www/catalog`.
3. Change to directory `/var/www/catalog`: `cd /var/www/catalog`. 
4. Clone the Item Catalog project from github:
   ```
   sudo git clone https://github.com/blueblackw/item-catalog.git catalog
   ```
   Now there is a new folder **catalog** in directory `/var/www/catalog`
5. Change to directory `/var/www`: `cd /var/www`.
6. From the `/var/www` directory, change the ownership of the `catalog` directory to user `grader`:
   ```
   sudo chown -R grader:grader catalog/
   ```
7. Change to the `/var/www/catalog/catalog` directory: `cd /var/www/catalog/catalog`.
8. Change the name of the **views.py** file to **__init__.py**: `mv views.py __init__.py`.
9. In __init__.py, at the end of the file find:
   ```
   app.run(host='0.0.0.0', port=5000)
   ```
   Change this line to: `app.run()`.

app.run()

   

   

   


