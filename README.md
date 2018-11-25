# Udacity - Linux Server Configuration Project
> Suha Baobaid

## Project Overview
This is the fifth project for the course [ FullStack Nanodegree by Udacity](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004 " FullStack Nanodegree by Udacity")

> You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

[ Catalog Item ](https://github.com/suhabaobaid/Item-Catalog-App), the third project in FSND is the app deployed to the server.

Main Source : [Digital Ocean](https://www.digitalocean.com/community/questions/best-practices-for-hardening-new-sever-in-2017) ; Best practices for strengthening a new server
Note: * indicates non-required/extra configurations
Important: SSH key is password protected and is provided with the private key

## Required Information
- Public IP address: 35.198.175.108
- Accessible SSH Port: 2200
- Application URL: http://35.198.175.108.xip.io/

## Configuration Walkthrough
### 1. Create Dev Environment: Create a Linux Virtual Machine & SSH into the server
Source: [ Google Cloud ](https://cloud.google.com/compute/docs/quickstart-linux) is chosen here as the provider
1.  Choose a provider (Compute Engine, Lighthouse, etc) and create a virtual mahcine instance, make sure to select Ubuntu 16.04 OS image.
2. Make note of the public IP address.
3. SSH into the VM instance using one of 3 options:
	* Through the CGP console
	* Using [GCLOUD SDK](https://cloud.google.com/compute/docs/instances/connecting-to-instance), which sets the metadata of the instance and generates SSH keys to be saved in your local machine.
	* Using [Third-party tools](https://cloud.google.com/compute/docs/instances/connecting-advanced)
	```
	ssh -i [PATH_TO_PRIVATE_KEY] [USERNAME]@[EXTERNAL_IP_ADDRESS]
	```

(Note): all the below commands are done in the VM unless stated otherwise
### 2. User Management: Create user named <b>grader</b> and provide sudo permission
1. Create a new user: `sudo adduser grader`
2. Provide sudo permission:
	* Create and edit a new file named grader under the sudoers.d dir `sudo nano /etc/sudoers.d/grader`
	* Add the following line and save the file: `grader ALL=(ALL:ALL) ALL`

### 3. Package Management: Update & upgrade packages
1. Update the current packages and the versions: `sudo apt update`
2. Install the newer versions of the packages: `sudo apt upgrade`
3. *Install [finger](http://manpages.ubuntu.com/manpages/xenial/man1/finger.1.html); user information lookup program: `sudo apt install finger`
4. *Include Cron scripts to automatically manage package updates:
	- Install unattended-upgrades package: `sudo apt install unattended-upgrades`
	- Enable the package: `sudo dpkg-reconfigure --priority=low unattended-upgrades`

### 4. Local time management:
Source: [ntp](https://www.digitalocean.com/community/tutorials/how-to-configure-ntp-for-use-in-the-ntp-pool-project-on-ubuntu-16-04)
1. `sudo dpkg-reconfigure tzdata`, opens the time configuration dialog; choose <i>None of the above</i> then <b>UTC</b>
2. Setup ntp for time synchronization: `sudo apt install ntp`

### 5. Key-Based Authentication & SSH Port number Change
1. Configure key-based authentication for <b>grader</b> user:
	1. <b>In your local machine</b> generate an encryption key, replace <i>KEY_FILENAME</i> with the name you want to rename your key files and <i>USERNAME</i> the user that would use this key; <b>grader</b> in this case: `ssh-keygen -t rsa -f ~/.ssh/[KEY_FILENAME] -C [USERNAME]` and restrict write on the file in case anyone gets hold of it: `chmod 400 ~/.ssh/[KEY_FILENAME]`
	2. Copy the content of <i>[KEY_FILENAME].pub</i> from your local machine
	3. In the VM (as root) create the file to keep the public key: `sudo nano /home/grader/.ssh/authorized_keys` paste the content copied and save the file.
	4. Restrict operations on the ssh and authorised keys:
	```
	sudo chmod 700 .ssh
	sudo chmod 644 .ssh/authorized_keys
	```
	5. Change the ownership to <b>grader</b>: `sudo chown -R grader:grader /home/grader/.ssh`
	6. Login to the VM instance with grader user using ssh: `ssh -i ~/.ssh/[KEY_FILENAME] grader@35.198.175.108`

2. Enforce SSH authentication, turn off root remote login & port number change: `sudo nano /etc/ssh/sshd_config`:
	- edit <b>PasswordAuthentication</b> value from <b>yes</b> to <b>no</b>
	- edit <b>PermitRootLogin</b> value from <b>without-password</b> to <b>no</b>
	- edit <b>Port</b> value from <b>22</b> to <b>2200</b>
	- Save the file
	- Restart the ssh service: `sudo service ssh restart`
	- Login to the VM using the new assigned port: `ssh -i ~/.ssh/[KEY_FILENAME] grader@35.198.175.108 -p 2200`

### 6. Configure the Uncomlicated Firewall (UFW)
Requirements states SSH (port 2200), HTTP (port 80) and NTP (port 123)
1. Disable all incoming and allow outgoing connections (considered good practice):
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
2. Allow the rules for the firewall:
	- `sudo ufw allow ssh`
	- `sudo ufw allow 2200/tcp`
	- `sudo ufw allow 80/tcp`
	- `sudo ufw allow 123/udp` -> this is needed for ntp
	- `sudo ufw enable`
3. To check the status of the ufw: `sudo ufw status`

### 6. * Login and attack management
Source: [Fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page)
Configure the firewall to monitor for repeated unsuccessful login attempts, ban attackers and send log email to the admin user
1. Install fail2ban: `sudp apt install fail2ban`
2. Create a new file to customize fail2ban functionality: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
3. Edit jail.local: `sudo nano /etc/fail2ban/jail.local`, set the destemail to the admin's email.
4. Install needed package to send emails: `sudo apt install sendmail`
5. Start the service: `sudo service fail2ban start`

### 7. Apache and mod_wsgi management
Install Apache to serve a python mod_wsgi app
1. Install Apache: `sudo apt install apache2`
2. Open your browser with your <i>Public IP</i>, it should display Apache default page
3. Install mod_wsgi and python-dev: `sudo apt install libapache2-mod-wsgi python-dev`
4. Enable mod_wsgi: `sudo a2enmod` wsgi
5. Restart Apache server to mod_wsgi to work: `sudo service apache2 restart`

### 8. Git management
1. Install Git: `sudo apt install git`
2. Set the dafault config:
	- set your username: `git config --global user.name [USERNAME]`
	- set your email: `git config --global email [EMAIL]`

### 9. Catalog app management
1. Create a directory catalog under in the category <i>/var/www</i>: `cd /var/www` then `sudo mkdir catalog`
2. Clone the Catalog app repo into the newly created dir: `cd /catalog` then `git clone https://github.com/suhabaobaid/Item-Catalog-App.git catalog`
3. Change the branch of the Catalog app to deployment as it contains adjustment to prepare it: `cd /catalog` then `sudo git checkout deployment`
4. Create <i>catalog.wsgi</i> file to serve the app over mod-wsgi:
`cd ..` then `sudo nano catalog.wsgi` and paste the following:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```

### 10. Virtual environment management
Source: [pip](https://www.saltycrane.com/blog/2010/02/how-install-pip-ubuntu/),
[venv](https://docs.python-guide.org/dev/virtualenvs/)
1. Install pip: `sudo apt install python-pip build-essential`
2. Install virtual environment to isolate and manage the executables needed: `sudo pip install virtualenv`
3. Create a virtual environment called <i>venv</i> (make sure you are in the directory <b>/var/www/catalog</b>): `sudo virtualenv venv`
4. Change permission to the virtual environment: `sudo chmod -R 777 venv`
5. Activate the virtual environement: `source venv/bin/activate`
6. Install app dependencies:
	- Flask: `pip install Flask`
	- Other dependencies: `pip install httplib2 sqlalchemy oauth2client requests python-psycopg2`
7. deactivate venv: `deactivate`

8. Configure a new Virtual Host:
	- Create a virtual host config file: `sudo nano /etc/apache2/sites-available/catalog.conf` and paste the following
	```
	<VirtualHost *:80>
      ServerName 35.198.175.108
      ServerAlias 35.198.175.108.xip.io
      ServerAdmin admin@35.198.175.108
      WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
      WSGIProcessGroup catalog
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
      <Directory /var/www/catalog/catalog/>
          Order allow,deny
          Allow from all
      </Directory>
      Alias /static /var/www/catalog/catalog/static
      <Directory /var/www/catalog/catalog/static/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  	</VirtualHost>
	```
	- Enable the virtual host: `sudo a2ensite catalog`
	- Restart Apache: `sudo service apache2 restart`

### 10. App repo access management
Make the Catalog app repository inaccessible from the web
1. Create .htaccess file in <i>/var/www/catalog</i> dir: `sudo nano .htaccess`
2. Paste the following: `RedirectMatch 404 /\.git`

### 11. Database management
Install and configure PostgreSQL
Note: '#' refers to the db prompt
Source: [PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)

1. Install PostgreSQL: `sudo apt install postgresql postgresql-contrib`
2. Connect to the db using the default created user: `sudo su - postgres` then access a Postgres prompt: `psql`
3. Create a new user called catalog with a preferred [PASSWORD]: `# CREATE USER catalog WITH PASSWORD '[PASSWORD]';`
4. Give permission for database creation: `# ALTER USER catalog CREATEDB;`
5. Create a database called <i>catalog</i>: `# CREATE DATABASE catalog WITH OWNER catalog;`
6. Connect to the created database, revoke all rights and grant access to the catalog user only and log out:
```
# \c catalog
# REVOKE ALL ON SCHEMA public FROM public;
# GRANT ALL ON SCHEMA public TO catalog;
# \q
```
7. Exit the db: `exit`
8. Change the connection in the Catalog app to the database using the created [PASSWORD]
9 Create the database schema: `python /var/www/catalog/catalog/setup_database.py`

### 12. OAuth management
1. Add your domain to Google's project authorized domains' list
2. Set the authorization uri and callback to the new url (include the public IP) for the project in Google's console

### 13. Running the application
1. Restart Apache: `sudo service apache2 restart`
2. Open the browser and enter your public IP
3. To check for error logs use: `sudo tail -20 /var/log/apache2/error.log`

<b>Special Thanks to [Stueken](https://github.com/stueken/FSND-P5_Linux-Server-Configuration) for a helpful and an elaborate README</b>
